# URLSession
*An object that coordinates a group of related network data transfer tasks.
SDKs*

`class URLSession : NSObject`

`URLSession` class provides an API
- for **downloading and uploading** data from/to **endpoints** indicated by **URLs**. 
- To enable your app to perform background downloads when your app isn’t running or while your app is suspended

A rich set of delegate methods 
- support authentication 
- allow your app to be notified of events like redirection.

> **What Is API?**
API: Application Programming Interface
API is like **messenger** that takes *requests* and tell the system what user want to do. Then, takes back the *responses* to you.
e.g. Waiter in Restaurant.


---
## The First Steps Before You read URLSession
---
### URL Loading System
*Interact with **URLs** and communicate with **servers** using standard Internet protocols.*
[ Document Link]("https://developer.apple.com/documentation/foundation/url_loading_system?changes=latest_minor")

**URL Loading System** provides **access to resources** identified by URLs, using standard protocols like `https`.
**Loading** is performed asynchronously
- your app can remain responsive
- your app can handle incoming data / errors as they arrive.

Can use **one** `URLSession`instance repeatedly to create `URLSessionTask`instances

`URLSessionTask` instance can
- fetch and return data to your app
- download / upload data & files to remote locations

`URLSessionConfiguration`: To configure a session
- how to use caches & cookies
- whether to allow connections on a cellular network

You can configure a session to run in the background, so that
While the app is suspended, the system can download data on its behalf & wake up the pp to deliver the results.

---

### Fetching Website Data into Memory
*Receive data directly into memory by creating a data task from a URL session*
[ Document]("https://developer.apple.com/documentation/foundation/url_loading_system/fetching_website_data_into_memory?changes=latest_minor")

Use `URLSessionDataTask` to receive response data into memory
<-> `URLSessionDownloadTask`(stored the data directly to the file system)

`URLsession` -> `URLSessionTask`
1. If your needs are simple, use `URLSession.shared`
Unless, create `URLSession` (like interacting with transfer through delegate callbacks)
2. Use `URLSessionConfiguration`instance
    - when creating a session
    - when passing in a class implemening `URLSessionDelegate`
3. After you have a session, create a data task with `dataTask()`
Tasks are created in a suspended state -> start: `resume()`

#### Receive Results: Completion Handler
*The simplest way to fetch data*
`URLSession` -> `URLSessionDdataTask` -> `CompletionHandler`

`URLSession.dataTask(with:)`: To create a data task
1. Verify that the `error == nil`. Handle the error and exit
2. Check `response` parameter to 
    - verify that the status code indicates success 
    - verify that the MIME type is an expected value
3. Use `data`instance as needed.

```Swift
func startLoad() {
    let url = URL(string: "https://www.example.com/")!
    let task = URLSession.shared.dataTask(with: url) { data, response, error in
        // Checking for local errors
        if let error = error { 
            self.handleClientError(error)
            return
        }
        
        // Checking for server errors
        guard let httpResponse = response as? HTTPURLResponse,
            (200...299).contains(httpResponse.statusCode) else {
            self.handleServerError(response)
            return
        }
        
        if let mimeType = httpResponse.mimeType, mimeType == "text/html",
            // Convert data to String
            let data = data,
            let string = String(data: data, encoding: .utf8) {
            DispatchQueue.main.async {
                // Populate a WKWebView outlet
                self.webView.loadHTMLString(string, baseURL: url)
            }
        }
    }
    task.resume()
}
```

> **Remeber:**
1. create dataTask
`URLSession.shared.dataTasks(with:)`
2. Check for local / server errors
`if let error = error {}`
`guard let httpResponse = response as? HTTPURLResponse`
3. Convert data to String and populate
`String(data:encoding:) { DispatchQueue.main.async { self.webView.loadHTMLString(_:baseURL:) }
}`


#### Receive Results: Completion Handler
`URLSession` -> `URLSessionDataTask`
`URLSession` -> `URLSessionDelegate`, `~TaskDelegate`, `~DataDelegate`

Portions of data are provided to
`URLSessionDataDelegate.Session(_:dataTask:didReceive:)`

Need to create own `URLSession`instance
1. Declare delegate protocols (`URLSessionDelegate`, `URLSEssionTaskDelegate`, `URLSessionDataDelegate`, `URLSessionDownloadDelegate`)
2. Create URL session with `init(configuration:delegate:delegateQueue:)`
```Swift
private lazy var session: URLSession = {
    let configuration = URLSessionConfiguration.default
    configuration.waitsForConnectivity = true
    return URLSession(configuration: configuration, delegate: self, delegateQueue: nil)
}()
```
> `waitsForConnectivity = true`: The session waits for suitable connectivity, rather than failing immediately

3. 
```Swift
var receivedData: Data?

func startLoad() {
    loadButton.isEnabled = false
    let url = URL(string: "https://www.example.com/")!
    receivedData = Data()
    let task = session.dataTask(with: url)
    task.resume()
}
```
Three delegate callbacks:
- `urlSession(_:dataTask:didReceive:completionHandler:)`     
     - verifies the response has a succesful HTTP status code
     - the MIME type is text/html or text/plain
     - If either of these is false, the task is canceled
-  `urlSession(_:dataTask:didReceive:)`
    - takes each `Data` instance received by the task
    - appends it to a buffer called `receivedData`
- `urlSession(_:task:didCompleteWithError:)`
    - first looks to see if a transport-level error has occurred.
    - If `error == nil`, it attempts to convert the `receivedData` buffer to a `String` and set it as the contents of `webView`.

```Swift
// delegate methods

func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive response: URLResponse,
                completionHandler: @escaping (URLSession.ResponseDisposition) -> Void) {
    guard let response = response as? HTTPURLResponse,
        (200...299).contains(response.statusCode),
        let mimeType = response.mimeType,
        mimeType == "text/html" else {
        completionHandler(.cancel)
        return
    }
    completionHandler(.allow)
}

func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive data: Data) {
    self.receivedData?.append(data)
}

func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?) {
    DispatchQueue.main.async {
        self.loadButton.isEnabled = true
        if let error = error {
            handleClientError(error)
        } else if let receivedData = self.receivedData,
            let string = String(data: receivedData, encoding: .utf8) {
            self.webView.loadHTMLString(string, baseURL: task.currentRequest?.url)
        }
    }
}
```





