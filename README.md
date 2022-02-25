```
URLSession, StarScream, Vapor + Fluent, Swift NIO, Swift Socket, BlueSocket

https://github.com/swiftsocket/SwiftSocket
https://www.raywenderlich.com/76-tcp-server-with-the-swiftnio-networking-framework
https://www.raywenderlich.com/books/server-side-swift-with-vapor/v3.0.ea1/chapters/29-websockets
https://www.raywenderlich.com/13209594-an-introduction-to-websockets 

Kitura

IBM wrote BlueSocket to serve as its low-level socket networking layer.

swift package manager :

swift package init --type=executable
swift package generate-xcodeproj 
swift build
swift package generate-xcodeproj 
```

```
mport Foundation

@available(iOS 13.0, *)
open class SocketServer: NSObject, URLSessionWebSocketDelegate {
    
    public var webSocket : URLSessionWebSocketTask? = nil

    public override init() {
        super.init()
        print("Socket Server")
        let session = URLSession(configuration: .default, delegate: self, delegateQueue: OperationQueue())
//      https://www.piesocket.com/websocket-tester
        let url = URL(string: "ws://localhost:8080/echo");
// wss://demo.piesocket.com/v3/channel_1?api_key=oCdCMcMPQpbvNjUIzqtvF1d2X2okWpDQj4AwARJuAgtjhzKxVEjQU6IdCjwm&notify_self
        webSocket = session.webSocketTask(with: url!);
    }

   open func start(){
//       webSocket?.receive()
       listen()
       webSocket?.resume()
       print("LISTENED")
    }
    
    func listen() {
        webSocket?.receive { [weak self] (result) in
          guard let self = self else { return }
          // 2
          switch result {
          case .failure(let error):
            print(error)
            return
          case .success(let message):
            print(message)
          }
          // 5
          self.listen()
        }
    }

   open func ping(){
        webSocket?.sendPing {
            error in if let error = error {
                print("Ping error: \(error)")
            }
        }
    }

   open func close(){
        webSocket?.cancel(with: .goingAway, reason: "Demo ended".data(using: .utf8))
    }

   open func send(){
        DispatchQueue.global().asyncAfter(deadline: .now()+1){
            self.webSocket?.send(.string("Send new message: \(Int.random(in: 0...1000))"),completionHandler: {
                error in if let error = error {
                    print("Send error: \(error)")
                }
                self.send()
            })
        }
    }

    open func receive(){
        webSocket?.receive(completionHandler: {[weak self] result in
            switch result {
            case .success(let message):
                switch message {
                case .data(let data):
                    print("Got Data: \(data)")
                case .string(let message):
                    print("Got String: \(message)")
                @unknown default:
                    break
                }
            case .failure(let error):
                print("Receive Error: \(error)")
            }
            self?.receive()
        }
        )
    }

    public func urlSession(_ session: URLSession, webSocketTask: URLSessionWebSocketTask, didOpenWithProtocol protocol: String?) {
            print("Socket Connected Establisehd")
        ping()
        receive()
        send()
    }

    public func urlSession(_ session: URLSession, webSocketTask: URLSessionWebSocketTask, didCloseWith closeCode: URLSessionWebSocketTask.CloseCode, reason: Data?) {
        print("Socket Connection Lost: \(String(describing: reason) )")
    }
}
// ServerViewModel.swift
// server.GET[“/ios/stream/offer”] = { _ in
      if #available(iOSApplicationExtension 13.0, *) {
        let socketServer = SocketServer()
        socketServer.start()
      }
      return HttpResponse.ok(.json(WebRTCViewModel().generatePayload().toJSON()), [“Content-Type” : “application/json”,“Access-Control-Allow-Origin”: “*”,“Access-Control-Allow-Headers”:“*”])
    }
```
