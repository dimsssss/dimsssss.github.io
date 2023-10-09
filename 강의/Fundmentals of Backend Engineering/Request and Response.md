- 클라이언트가 요청을 보낸다
- 서버는 요청을 파싱한다(파싱은 비용이 많이 드는 작업)
- 서버는 요청을 처리한다
- 서버는 응답을  보낸다
- 클라이언트는 요청을 파싱하고 사용한다

데이터가 큰 한 개의 요청 vs 데이터가 작은 여러 개의 요청

사람들이 초기에 xml을 사용했지만 파싱이 느려서  json으로 넘어갔고 마찬가지로 이 마저도 느려져서 proto buffer로 넘어감

### Where is used
- web, http, dns, ssh
- RPC(Remote Procedure Call)
- SQL and Database Protocol
- APIs(REST, GraphQL, SOAP)
- Implemented in variations
### Anatomy of a Request / Response
- A request structure is defined by both client and server
- Request has a boundary
- Defined by a protocol and message format
- Same for the response
- E.g. HTTP Request
### Building and upload image service with request / response
- Send large request with the image(simple)
- Chunk image and send a request per chunk(resumable)
### Doesn't work everywhere
- Notification Service
- Chatting application
- Very Long requests