### sse가 뭘까?
sse-> server sent event 의 줄임말.
서버가 보내는 이벤트라는 건데 이 이벤트가 뭐냐면 간단하게 말해서 
일반적으로 요청을 한번 보내고, 응답을 받으면 연결이 끝나는데 
sse는 한 번 연결하면 서버가 필요할 때마다 계속 데이터를 보낼 수 있는것.

### 왜 필요할까?
일반적으로 한 번 요청하면 다시 요청해야 하는데, sse를 안 쓰면 실시간 알림 기능 등 구현에서는 폴링으로 요청을 지속적으로 보냄으로서 필요 없는 요청이 막 보내지는데.

근데 사용하게 된다면? 처음 한 번만 요청하면 연결된 상태가 유지되니까 실시간성이 증가한다. + 한 번 연결하면 서버가 필요할 때마다 계속 데이터를 보낼 수 있다.

연결 한 이후에 새 알림이 발생하면 서버가 바로 보내줌
### 예제 코드

```
@RestController
@RequestMapping("/api/sse")
public class SseController {

    // 클라이언트가 연결을 유지할 목록
    private final List<SseEmitter> emitters = new CopyOnWriteArrayList<>();

    @GetMapping(value = "/subscribe", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public SseEmitter subscribe() {
        SseEmitter emitter = new SseEmitter(60 * 1000L); // 타임아웃 60초
        emitters.add(emitter);

        emitter.onCompletion(() -> emitters.remove(emitter));
        emitter.onTimeout(() -> emitters.remove(emitter));
        emitter.onError((e) -> emitters.remove(emitter));

        // 연결 확인용 첫 이벤트
        try {
            emitter.send(SseEmitter.event()
                    .name("connect")
                    .data("연결 성공!"));
        } catch (IOException e) {
            emitters.remove(emitter);
        }

        return emitter;
    }

    // 다른 API에서 호출해서 모든 클라이언트에 메시지 전송
    public void sendToAll(String message) {
        List<SseEmitter> deadEmitters = new ArrayList<>();
        emitters.forEach(emitter -> {
            try {
                emitter.send(SseEmitter.event()
                        .name("message")
                        .data(message));
            } catch (IOException e) {
                deadEmitters.add(emitter);
            }
        });
        emitters.removeAll(deadEmitters);
    }

    // 테스트용: 이 엔드포인트 호출하면 위에서 연결된 클라이언트들한테 방송
    @PostMapping("/broadcast")
    public void broadcast(@RequestParam String message) {
        sendToAll(message);
    }
}
``` 
