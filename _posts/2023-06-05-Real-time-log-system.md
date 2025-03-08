---
title:  Real-time log system with websocket
categories:
  - CI/CD
tags:
  - Loki
  - Grafana
  - Prometheus
  - Websocket
  - Redis
  - Project experience

---

Content

{% include toc %}

# 1. Background

## 1.1 Overview

The user-side task log has a real-time query button. The front-end and web-api use websocket communication, and the web-api and loki also use websocket communication (/tail interface). That is, the web-api layer serves as the server of the front-end websocket connection and the client of the back-end loki websocket connection.

## 1.2 Online issues

Online bug: http://10.0.1.125/browse/GEMACC-29

[ http://10.0.1.125/browse/GEMCLOUD-235](http://10.0.1.125/browse/GEMCLOUD-235)

# 2. Problem analysis

There are two core issues currently exposed online:

1. The backend requests loki and reports an error, but the frontend is unaware, so after the user turns on the real-time log, the page does not change;

2. The backend and frontend detection will cause data race problems;

After analysis, the main reasons are:

1. The interaction with the frontend does not transmit error information or shutdown signals, resulting in the frontend page not changing, and the user does not know what went wrong;

2. By checking the logs, it was found that the websocket long connection between loki/tail was abnormally disconnected many times, so a reconnection mechanism is needed;

3. Another possible problem is that the web-api heartbeat and normal information transmission generate data race and report an error, and the front-end connection is not closed in time;

# 3. Technical solution

## 3.1 Main functions

### 3.1.1. Connection limit

Storage in redis: Use the operator ID as the key value, and limit each operator to open only 10 websocket connections.

### 3.1.2. Listen to front-end information

Individual goroutine receives information from the front-end, and generally only close information is available;

If the information reading is abnormal, the connection with the front-end may be abnormal, and an exception error may be thrown;

After receiving the above two types of information, directly notify the back-end main function and other coroutines to exit;

### 3.1.3. Establish a long connection with the back-end loki to obtain log information

Individual goroutine, the main function notifies the coroutine through the channel, and the goroutine notifies the main function through the pipeline through internal error reporting;

In the goroutine that queries loki, it is divided into the main function and the goroutine that queries loki. The main function is responsible for controlling sub-coroutine errors and receiving external main function closing information;

### 3.1.4 The long connection with the back-end loki implements a reconnection mechanism

If the long connection between web-api and loki is abnormally closed, five retries are required, and the time interval between each retry is between 5-10s. If the function still fails after five retries, an error is thrown directly, and the coroutine notifies the external main function to exit through the pipeline. The external main function disconnects from the front end and passes the error message.

Refer to the implementation in grafana logcli: https://github.com/grafana/loki/blob/main/pkg/logcli/query/tail.go

```golang
import "github.com/grafana/dskit/backoff"

for {
 err := unmarshal.ReadTailResponseJSON(tailResponse, conn)
 if err != nil {
 // Check if the websocket connection closed unexpectedly. If so, retry.
 // The connection might close unexpectedly if the querier handling the tail request
 // in Loki stops running. The following error would be printed:
 // "websocket: close 1006 (abnormal closure): unexpected EOF"
 if websocket.IsCloseError(err, websocket.CloseAbnormalClosure) {
 log.Printf("Remote websocket connection closed unexpectedly (%+v). Connecting again.", err)

 // Close previous connection. If it fails to close the connection it should be fine as it is already broken.
 if err = conn.WriteMessage(websocket.CloseMessage, websocket.FormatCloseMessage(websocket.CloseNormalClosure, "")); err != nil {
 log.Printf("Error closing websocket: %+v", err)
 }

 // Try to re-establish the connection up to 5 times.
 backoff := backoff.New(context.Background(), backoff.Config{
 MinBackoff: 1 * time.Second,
 MaxBackoff: 10 * time.Second,
 MaxRetries: 5,
 })

 for backoff.Ongoing() {
 conn, err = c.LiveTailQueryConn(q.QueryString, delayFor, q.Limit, lastReceivedTimestamp, q.Quiet)
 if err == nil {
 break
 } log.Println("Error recreating tailing connection after unexpected close, will retry:", err)
 backoff.Wait()
 }

 if err = backoff.Err(); err != nil {
 log.Println("Error recreating tailing connection:", err)
 return
 }

 continue
 }

 log.Println("Error reading stream:", err)
 return
 }

 labels := loghttp.LabelSet{}
 for _, stream := range tailResponse.Streams {
 if !q.NoLabels {
 if len(q.IgnoreLabelsKey) > 0 || len(q.ShowLabelsKey) > 0 {

 ls := stream.Labels

 if len(q.ShowLabelsKey) > 0 {
 ls = matchLabels(true, ls, q.ShowLabelsKey)
 } if len(q.IgnoreLabelsKey) > 0 {
 ls = matchLabels(false, ls, q.ShowLabelsKey)
 }

 labels=ls

 } else {
 labels = stream.Labels
 }
 }

 for _, entry := range stream.Entries {
 out.FormatAndPrintln(entry.Timestamp, labels, 0, entry.Line)
 lastReceivedTimestamp = entry.Timestamp
 }

 }
 if len(tailResponse.DroppedStreams) != 0 {
 log.Println("Server dropped following entries due to slow client")
 for _, d := range tailResponse.DroppedStreams {
 log.Println(d.Timestamp, d.Labels)
 }
 }
 }
```



### 3.1.5 Heartbeat with front-end

Use multiplexing to listen to the time.ticker timer and send pingMessage to the front-end every second. If the heartbeat fails, it means that the connection with the front-end is abnormal, and the main function will be notified to exit and close the connection with the front-end, and the back-end loki long connection will also be notified to exit;

Here, timer multiplexing is used instead of starting a heartbeat coroutine alone. If a separate go routine sends heartbeat information to the front-end, normal data and heartbeat data will need to be written to the same pipeline at the same time, which will cause data race problems.

Refer to loki/pkg/querieer/http.go TailHandler(), https://github.com/grafana/loki/blob/main/pkg/querier/http.go, to prevent multiple goroutines from writing messages to the writer at the same time, causing data race.

```golang
for {
 select {
 case response = <-responseChan:
 var err error
 if loghttp.GetVersion(r.RequestURI) == loghttp.VersionV1 {
 err = marshal.WriteTailResponseJSON(*response, conn)
 } else {
 err = marshal_legacy.WriteTailResponseJSON(*response, conn)
 }
 if err != nil {
 level.Error(logger).Log("msg", "Error writing to websocket", "err", err)
 if err := conn.WriteMessage(websocket.CloseMessage, websocket.FormatCloseMessage(websocket.CloseInternalServerErr, err.Error())); err != nil {
 level.Error(logger).Log("msg", "Error writing close message to websocket", "err", err) }
 return
 }

 case err := <-closeErrChan:
 level.Error(logger).Log("msg", "Error from iterator", "err", err)
 if err := conn.WriteMessage(websocket.CloseMessage, websocket.FormatCloseMessage(websocket.CloseInternalServerErr, err.Error())); err != nil {
 level.Error(logger).Log("msg", "Error writing close message to websocket", "err", err)
 }
 return
 case <-ticker.C:
 // This is to periodically check whether connection is active, useful to clean up dead connections when there are no entries to send
 if err := conn.WriteMessage(websocket.PingMessage, nil); err != nil {
 level.Error(logger).Log("msg", "Error writing ping message to websocket", "err", err)
 if err := conn.WriteMessage(websocket.CloseMessage, websocket.FormatCloseMessage(websocket.CloseInternalServerErr, err.Error())); err != nil {
 level.Error(logger).Log("msg", "Error writing close message to websocket", "err", err)
 }
 return
 }
 case <-doneChan:
 return }
}
```

Use case to implement multiplexing. Only one pipeline will write data to c.writer at a time. Other pipelines will be blocked after receiving the information. They will write data to c.writer after the next allocation, avoiding data competition.

### 3.1.6 Set expiration time

Set the expiration time through contex, and then use multiplexing to listen to the expiration signal to limit the total connection time.

```golang
contex, cancel := context.WithTimeout(context.Background(), time.Hour*24)
defer func() {
cancel()
}()
for {
// Listen for expiration signals, default 24h
case <-contex.Done():
// Close loki websocket long connection
if lokiConn != nil {
if err := lokiConn.WriteMessage(websocket.CloseMessage, websocket.FormatCloseMessage(websocket.CloseNormalClosure, "")); err != nil {
logsCtx.Infof("Error writing close message to websocket: %v", err)
}
}
logsCtx.Info("Loki tail connection timeout")
return errors.WithStack(apierr.ErrInvalidData().SetCause(err).SetReason("Loki tail connection timeout"))
}
}
```

## 3.2 Notification communication method between various functional modules

![real-time_log](https://raw.githubusercontent.com/infinityjay/myImageHost/main/typora/real-time_log.png)

## 3.3 Front-end changes

### 3.3.1 Modify the request method

Previous solution: first establish a connection, and then pass the request parameters in the pipeline.

Problem: Token needs to be verified before establishing a connection; this request method is not standard enough. The front end generally transmits shutdown information in the pipeline and should not transmit other information.

After optimization: Request parameters are added to the query.

Interface document: Real-time query of task log [websocket interface push] GET /user/websocket/job/log Interface ID: 60068599 Interface address: https://www.apifox.cn/web/project/441541/apis/api-60068599

Request example:

[wss://10.10.100.147:31443/gemini/v1/gemini_api/gemini_api/user/websocket/job/log?instanceId=71881&startTime=1677577273 146&bearerToken=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjEsInVzZXJOYW1lIjoiMTU4ODg4ODg 4ODgiLCJ1c2VyUm9sZUlkcyI6WzFdLCJleHAiOjE2Nzc4MzQ5NjQsImlzcyI6ImdlbWluaS11c2VyYXV0aCJ9.mG-uYsUmEb ZO2r_xH_oMZDhG5rhK_XKZL0RvaQz2kXg](wss://10.10.100.147:31443/gemini/v1/gemini_api/gemini_api/use r/websocket/job/log?instanceId=71881&startTime=1677577273146&bearerToken=eyJhbGciOiJIUzI1NiIsInR 5cCI6IkpXVCJ9.eyJ1c2VySWQiOjEsInVzZXJOYW1lIjoiMTU4ODg4ODg4ODgiLCJ1c2VyUm9sZUlkcyI6WzFdLCJleHA iOjE2Nzc4MzQ5NjQsImlzcyI6ImdlbWluaS11c2VyYXV0aCJ9.mG-uYsUmEbZO2r_xH_oMZDhG5rhK_XKZL0RvaQz2kXg)

### 3.3.2 Optimize request processing

The current online situation is that the page remains unchanged after the user clicks on the real-time log;

After receiving the backend close message, the frontend needs to actively close the connection, let the user know, and prompt an error or retry later;

### 3.3.3 Reconnection mechanism (specific settings of the frontend)

The frontend needs to implement the backend detection and reconnection mechanism. In addition to the normal closing information, other abnormal disconnections due to network jitter need to be reconnected. It is recommended not to set the number of reconnections and intervals too large. If the reconnection fails, directly prompt the user to retry later;

* Backend return code comparison table

| Status code | Meaning | Whether to reconnect | Whether to prompt the user to retry later | Remarks |
| ---------- | -------------------------- | -------------- | ------------------------ | ------------------------------------------------------------ |
| 1000 | Normal closing information | No | No | |
| 1007 | Request parameter error | No | No | Generally only appears during joint debugging; |
| 999 | (Custom) token verification failed | Yes | No | It may be that the token has expired, and the latest token needs to be obtained from the cookie; |
| 1008 | The number of connections has reached the upper limit (10) | No | Yes | The text remains the same, and it is generally difficult to trigger. Prompt the user to close the unused real-time logs and try again; |
| 1011 | Backend internal service error | Yes | Yes | It may be a write exception, or an abnormal connection with loki. You can try to reconnect first. If the reconnection fails, the text can prompt the user to try again later; |
| Other status codes | It may be an abnormal disconnection, etc. | No | Yes | It may be a network jitter or other abnormalities. The text can prompt the user to try again later; |

### 3.3.4 Set the timeout time

This can be considered. Do you need to actively close after not obtaining information for a period of time, and then let the user open it by himself?

# 4. Test focus

## 4.1 Abnormal disconnection between front-end and web-api

Abnormal disconnection between front-end and web-api requires testing:

- The front-end needs to actively prompt the user to try again later;
- The back-end needs to disconnect (you can see the web-api log);
- You need to check whether the back-end goroutine exits;

## 4.2 Abnormal disconnection between web-api and loki

- First, you need to construct the network jitter between the web-api and loki to disconnect the websocket long connection;
- After disconnection, web-api will retry loki every 5-10s, for a total of 5 times (you can see the web-api log);
- If the retry is successful, the front-end should not be aware of it. When the retry is successful, the log is output normally;
- If the retry fails, the back-end needs to exit, monitor whether there is a goroutine leak, and check whether the front-end prompts the user to report an error or retry later;

## 4.3 Connection limit

Due to code changes, it is necessary to test and verify that each operator can only open 10 connections. If the number of connections exceeds 10, an error will be reported.

## 4.4 Concurrency test

It is necessary to test and do a high-concurrency stress test, observe the memory and goroutine number, and then turn off the stress test to check whether the memory and goroutine decrease normally.

## 4.5 Some flag logs in web-api

Can be uniformly filtered with job/log.go

- Coroutine A exits

"Unexpected error from client: %+v"

"Websocket client abnormal close: %+v"

"Websocket client normal close: %+v"

- Coroutine B exits

"Cancel from websocket connection"

"Loki tail connection timeout"

job/log.go 769 lines

- Coroutine C exits

"Loki tail connection closed"

- Main process exits

"Total websocket connection count : %v"

# 5. Other issues

- [ ] Can we directly forward at the ingress layer and let the front end directly request the loki/tail interface;

Example of requesting the loki tail interface [ws://10.10.100.147:30200/loki/api/v1/tail?limit=30&query=%7Bfilename%3D%22%2Fvar%2Flog%2Fgemini%2F1%2Fdbbdf9da0091925ced09facb9 abfb2fb%2Ftaskrole1%2F79a27998-2803-4fb1-a207-760f72b79f62%2Fuser.gemini.all%22%7D&start=1678102568153528700](ws://10.10.100.147:30200/loki/api/v1/tail?limit=30&query={filename%3D"%2Fvar%2Flog%2Fgemini%2F1%2Fdbbdf9da0091925ced09facb9abfb2fb%2Ftaskrole1%2F79a27998-2803-4fb1-a207-760f72b79f62%2Fuser.gemini.all"}&start=1678102568153528700)

The connection limit and query conditions can be quickly returned through an interface, and the authentication problem is not easy to solve.

- [ ] The test needs to be concurrent;

