# 과제 요구사항 (필수)

### Lv 1. Docker로 MySQL과 Redis 설정 `필수`

- [ ] Docker로 MySQL과 Redis를 실행합니다.
- [ ] Spring 애플리케이션의 환경 변수를 설정합니다.

### Lv 2. SQL을 JPA 인덱스로 표현하기 `필수`

구현 위치: `chat/entity/ChatMessage`

다음 SQL은 월드별 최근 채팅 조회에 사용할 인덱스를 생성합니다. 이 SQL과 같은 인덱스가 생성되도록 `ChatMessage`의 `@Table`을 수정하세요.

```sql
CREATE INDEX idx_chat_world_created_at ON chat_messages(world_id, created_at);
```

> `@Table`의 `indexes`와 `@Index`의 `name`, `columnList` 사용법을 확인해 보세요.

- [ ] SQL을 직접 실행하는 대신 `@Table`과 `@Index`로 인덱스를 선언합니다. 테이블 이름, 인덱스 이름과 컬럼 순서는 제공된 SQL과 같아야 합니다.
- [ ] **확인:** 서버를 실행하고 인덱스가 실제 DB에 생성됐는지 확인합니다.
- [ ] **확인:** 인덱스가 없으면 시작 검사에서 서버 실행을 중단합니다. `CHAT_HISTORY_INDEX_MISSING` 오류가 사라지고 서버가 정상 실행되어 `http://localhost:8080/`에서 첫 화면이 열립니다.

### Lv 3. 요청 검증과 DTO: 플레이어 등록 `필수`

구현 위치: `player/controller/PlayerController`, `player/dto/CreatePlayerRequest`, `player/service/PlayerService`

**API 명세 → 플레이어 등록**

- [ ] API 명세에 맞게 플레이어 등록 Controller, 요청 DTO와 서비스를 구현합니다. 닉네임은 비어 있지 않은 2~12글자이며, 영문 대소문자와 숫자, 밑줄만 허용합니다.
  - 정규식: `^[a-zA-Z0-9_]+$`
- [ ] 이미 등록된 닉네임이면 `ConflictException`으로 `DUPLICATE_NICKNAME` 에러를 던집니다.
- [ ] Controller의 요청 매핑, JSON 본문 바인딩, DTO 검증과 성공 응답을 명세대로 구현합니다.
- [ ] 중복이 아니면 제공된 `savePlayer(new Player(request.getNickname()))`로 저장합니다. 동시 등록의 제약 위반 처리는 이 함수에서 제공합니다. 성공 응답은 명세대로 본문 없는 `201`입니다.
- [ ] **테스트 확인:** `PlayerRegistrationTest.java`와 `PlayerApiTest.java`의 주석을 해제한 뒤 실행합니다.
- [ ] **확인:** 게임 화면에서 닉네임을 적용할 수 있습니다. 잘못된 닉네임이나 이미 등록된 닉네임은 새로 저장되지 않습니다.

### Lv 4. 월드 생성 `필수`

구현 위치: `world/service/WorldService`

**API 명세 → 월드 목록, 월드 생성**

- [ ] `worldOperations.duringCreation()`에 람다를 전달하고 그 결과를 반환합니다. 이 함수는 동시에 들어온 생성 요청이 순서대로 처리되도록 합니다.
- [ ] 람다 안에서 `worldRepository.countRootWorlds()`가 `MAX_WORLDS` 이상이면 `ConflictException`으로 `WORLD_LIMIT_REACHED` 에러를 던집니다.
- [ ] 제한을 넘지 않으면 `createPreparedWorld(request)`의 결과를 반환합니다.
- [ ] **테스트 확인:** `WorldCreationTest.java`의 주석을 해제한 뒤 실행합니다.
- [ ] **확인:** 새 월드가 목록에 표시되고 서버를 재시작해도 남아 있습니다.

### Lv 5. 채팅 저장과 내역 조회 `필수`

구현 위치: `chat/service/ChatService`

**API 명세 → 최근 채팅 조회, `GET /worlds/{worldId}/chats?limit=50`**

- [ ] 제공된 채팅 핸들러에서 호출할 저장 서비스를 구현합니다. 보낸 사람은 메시지 본문이 아니라 연결에 저장된 사용자 정보로 결정합니다. 저장 결과는 제공된 `savedResponse(worldId, saved)`로 반환합니다. 이 함수는 응답 생성과 저장 완료 이벤트 발행을 담당합니다.
- [ ] 채팅 저장에는 `chatMessageRepository.save()`를 사용합니다.
- [ ] 최근 채팅을 조회하는 코드는 아래와 같습니다.

```java
List<ChatMessage> recent = chatMessageRepository.findByWorldIdOrderByCreatedAtDescIdDesc(worldId, PageRequest.of(0, capped));
```

- [ ] **테스트 확인:** `ChatServiceTest.java`의 주석을 해제한 뒤 실행합니다.
- [ ] **확인:** 저장된 내용과 반환된 목록의 건수 및 순서를 확인합니다. 이 검사는 게임 서버나 REST API 실행 없이 수행합니다.

### Lv 6. 최근 채팅 조회 API 구현 `필수`

구현 위치: `chat/controller/WorldChatController`

**API 명세 → 최근 채팅 조회**

- [ ] 요청 경로, HTTP 메서드, 경로 변수와 선택 파라미터의 기본값을 명세에 맞게 구현합니다.
- [ ] 제공된 `RecentChatQueryService.getRecentMessages()`의 결과를 `ResponseEntity`에 담아 반환합니다.
- [ ] **테스트 확인:** `RecentChatApiTest.java`의 주석을 해제한 뒤 실행합니다.
- [ ] **확인:** API를 호출하여 성공 상태 코드와 응답을 확인합니다. 아직 채팅을 저장하지 않았다면 빈 배열이 정상입니다. 저장된 채팅이 있다면 응답 필드와 순서도 확인합니다.

### Lv 7. WebSocket 연결과 사용자 식별 `필수`

구현 위치: `ws/NicknameHandshakeInterceptor`

**API 명세 → WebSocket 연결**

연결 주소는 `ws://localhost:8080/ws/worlds/{worldId}?nickname={nickname}`입니다. 실제 등록한 닉네임과 생성한 월드 ID를 사용합니다.

`beforeHandshake()`의 TODO 세 곳을 구현합니다.

- [ ] `playerRepository.findByNickname(nickname)`으로 플레이어를 조회해 `player`에 대입합니다. 조회 결과가 없으면 `null`을 사용합니다.
- [ ] `worldRepository.findById(worldId)`로 월드를 조회해 `world`에 대입합니다. 조회 결과가 없으면 `null`을 사용합니다.
- [ ] `attributes`에 `ATTR_NICKNAME`을 키로 `nickname`을, `ATTR_WORLD_ID`를 키로 `worldId`를 저장합니다. 이 값은 연결 이후 `WebSocketSession.getAttributes()`에서 사용할 수 있습니다.
- [ ] **테스트 확인:** `NicknameHandshakeInterceptorTest.java`의 주석을 해제한 뒤 실행합니다.
- [ ] **확인:** 제공 테스트에서 정상 요청의 닉네임과 월드 ID가 세션 속성에 저장되는지 확인합니다.

### Lv 8. HandshakeInterceptor 등록 `필수`

구현 위치: `config/WebSocketConfig`

`NicknameHandshakeInterceptor` 클래스는 존재하지만 WebSocket 설정에 등록되지 않은 상태입니다. 클래스가 Spring Bean으로 등록되어 있어도 연결 요청에 자동으로 적용되지는 않습니다.

- [ ] `WebSocketConfig`에서 `/ws/worlds/{worldId}` 경로에 `NicknameHandshakeInterceptor`를 등록합니다. 인터셉터를 새로 만들거나 사용자 식별 로직을 핸들러로 옮기지 않습니다.
- [ ] **테스트 확인:** `WebSocketConfigTest.java`의 주석을 해제한 뒤 실행합니다.
- [ ] **확인:** Postman으로 연결을 요청하고 로그 또는 디버거로 `beforeHandshake()` 실행과 월드 ID 및 닉네임의 세션 속성 저장을 확인합니다.

### Lv 9. 월드별 WebSocket 세션 관리 `필수`

구현 위치: `ws/WorldSessionRegistry`

현재 서버의 WebSocket 연결을 보관하도록 `register()`와 `get()`을 구현합니다.

- [ ] `register()`에서 `sessions.putIfAbsent(nicknameKey, candidate)`로 연결을 등록합니다. 반환값이 `null`이면 새로 등록한 것이므로 `added`를 `true`로 설정합니다. 이미 등록된 연결이 있으면 덮어쓰지 않습니다.
- [ ] `get()`에서 `sessions.get(key(nickname))`으로 연결을 조회해 반환합니다.
- [ ] **테스트 확인:** `WorldSessionRegistryTest.java`의 주석을 해제한 뒤 실행합니다.
- [ ] **확인:** 등록한 연결을 월드와 닉네임으로 조회할 수 있고, 같은 월드의 중복 닉네임은 기존 연결을 덮어쓰지 않습니다.

### Lv 10. Redis 접속 상태 관리 `필수`

구현 위치: `presence/PresenceService`

Redis에는 실제 소켓 객체 대신 연결 ID와 만료 시각을 저장합니다. `join()`과 `leave()`에서 Redis를 호출하는 한 줄씩만 구현합니다.

- Sorted Set의 **member**: `connectionId` — 연결을 구분하는 값입니다.
- Sorted Set의 **score**: `expiresAt()` — 해당 연결의 만료 시각입니다.
- 키: `key(worldId)` — `world:{worldId}:presence` 형식입니다.

- [ ] `join()`에서 `redisTemplate.opsForZSet().add(key, connectionId, expiresAt())`로 접속 정보를 저장합니다.
- [ ] `leave()`에서 `redisTemplate.opsForZSet().remove(key(worldId), connectionId)`로 종료된 연결을 삭제합니다.
- [ ] 연결별 만료는 90초, 키 전체 정리용 TTL은 180초입니다.
- [ ] **테스트 확인:** Docker를 실행하고 `PresenceServiceTest.java`의 주석을 해제한 뒤 실행합니다.
- [ ] **확인:** 연결하면 Redis에 연결 ID와 만료 시각이 저장되고, 정상 종료하면 해당 원소가 제거됩니다. `ZRANGE 키 0 -1 WITHSCORES`로 확인하세요.

### Lv 11. 메시지 라우팅과 Ping/Pong `필수`

구현 위치: `ws/MessageRouter`, `ws/handler/PingWsHandler`

**API 명세 → 메시지 공통 형식, ping 요청, pong 응답**

- [ ] `MessageRouter.route()`에서 찾아 둔 `handler`의 `handle(context, message)`를 호출합니다.
- [ ] `PingWsHandler`에서 `presenceService.heartbeat(context.worldId(), connection.connectionId())`를 호출해 현재 연결의 Redis 접속 상태를 갱신합니다.
- [ ] `broadcaster.sendTo(context.session(), new PongResponse())`로 ping을 보낸 연결에 pong을 응답합니다. 15초마다 ping을 보내는 클라이언트 코드는 제공합니다.
- [ ] **테스트 확인:** `MessageRouterTest.java`와 `PingWsHandlerTest.java`의 주석을 해제한 뒤 실행합니다.
- [ ] **확인:** 게임에 입장한 상태에서 `GET /worlds`의 접속 인원이 90초 후에도 유지되는지 확인합니다. ping에 대한 pong 응답과 Redis 갱신 호출은 제공 테스트로 확인합니다.

### Lv 12. 플레이어 이동 요청 처리 `필수`

구현 위치: `ws/handler/MoveWsHandler`

**API 명세 → 플레이어 이동 요청**

연결은 유지되지만 서버에 이동 요청이 전달되지 않습니다. 명세의 요청 필드와 타입을 읽고 핸들러를 완성하세요.

- [ ] 제공된 `WsFields.finiteNumber()`, `finiteFloat()`, `booleanValue()`로 요청 값을 읽습니다. 각 메서드는 메시지와 필드명을 받습니다.
- [ ] `PlayerAction.Move`에 현재 연결의 닉네임, 위치, 시선과 이동 상태를 전달하고 `engineManager.enqueue(월드 ID, 이동 요청)`를 호출합니다. 월드와 닉네임은 요청 본문에서 받지 않습니다.
- [ ] `PlayerAction.Move`의 생성자 순서는 닉네임, x, y, z, yaw, pitch, crouching, gliding, 내부 식별자입니다. 내부 식별자 조회는 제공 코드 그대로 사용합니다.
- [ ] **테스트 확인:** `MoveWsHandlerTest.java`의 주석을 해제한 뒤 실행합니다.
- [ ] **확인:** 게임에서 이동 키를 눌러 자신의 캐릭터가 이동하는지 확인합니다.

### Lv 13. 채팅 요청 처리와 응답 구성 `필수`

구현 위치: `ws/handler/ChatWsHandler`, `ws/dto/ChatResponse`

**API 명세 → 채팅 요청, 채팅 응답**

게임의 채팅 메시지를 읽어 저장하고, 명세에 맞는 WebSocket 응답 객체를 구성하세요.

- [ ] `readContent()`에서 요청 내용을 읽습니다. 문자열 조회는 제공된 `WsFields.text(메시지, 필드명)`을 사용할 수 있습니다.
- [ ] `createResponse()`에서 현재 연결의 월드와 닉네임으로 저장 서비스를 호출하고 저장 결과를 응답 DTO로 변환합니다.
- [ ] 명세를 보고 `ChatResponse`의 필드와 생성자를 완성합니다.
- [ ] **테스트 확인:** `ChatWsHandlerTest.java`의 주석을 해제한 뒤 실행합니다.
- [ ] **확인:** 게임에서 일반 채팅을 보내고 최근 채팅 조회 API와 DB에서 저장 결과를 확인합니다. 채팅은 아직 본인과 다른 참여자의 화면에 표시되지 않습니다.

### Lv 14. 같은 월드의 참여자에게 채팅 전송 `필수`

구현 위치: `chat/service/LocalChatSender`

**API 명세 → 채팅 전송 응답**

채팅 응답은 제공된 `ChatDelivery`를 거쳐 `LocalChatSender.send(worldId, message)`로 전달됩니다. `worldId`와 `message`를 받아 같은 월드에 방송하는 부분만 구현하세요.

- [ ] `WorldBroadcaster.broadcast(worldId, message)`로 같은 월드의 세션에 메시지를 전달합니다. 보낸 사람도 수신 대상에 포함합니다.
- [ ] 연결별 전송 실패 처리는 제공된 `WorldBroadcaster`에 맡깁니다.
- [ ] **테스트 확인:** `LocalChatSenderTest.java`의 주석을 해제한 뒤 실행합니다.
- [ ] **확인:** 같은 월드의 두 참여자가 보낸 사람, 내용과 시각을 포함한 채팅을 받습니다. 다른 월드의 참여자에게는 전달되지 않으며 DB에는 보낸 채팅 한 건만 저장됩니다.

### Lv 15. 접속자 목록 조회 `필수`

구현 위치: `ws/handler/OnlineUsersWsHandler`, `ws/dto/OnlineUsersResponse`

**API 명세 → 접속자 목록 요청, 접속자 목록 응답**

현재 월드에 접속한 사용자를 WebSocket으로 조회하려고 합니다. 명세에 맞는 응답 DTO와 핸들러를 완성하세요.

- [ ] `registry.entries(월드 ID)`에서 열린 연결만 선택합니다. 닉네임은 각 세션의 `NicknameHandshakeInterceptor.ATTR_NICKNAME` 속성으로 확인합니다.
- [ ] 닉네임 목록을 명세의 기준대로 정렬해 `users`에 담고, 목록의 크기를 `count`에 담습니다. `broadcaster.sendTo()`로 요청한 연결에만 응답합니다.
- [ ] **테스트 확인:** `OnlineUsersWsHandlerTest.java`의 주석을 해제한 뒤 실행합니다.
- [ ] **확인:** 게임 창 하나만 남기고 나머지 연결을 종료합니다. 게임에서 사용 중인 닉네임과 다른 닉네임을 등록해 Postman으로 같은 월드에 연결합니다. 명세의 요청을 보내 두 닉네임과 인원수 `2`가 응답에 포함되는지 확인합니다. 게임 연결을 종료한 뒤 다시 요청하면 Postman의 닉네임만 남아야 합니다.

# 과제 요구사항 (도전)

### Lv 16. 낙관적 락 `도전`

구현 위치: `trial/entity/WorldTrialSite`

서로 같은 상태를 읽은 두 저장 작업이 겹쳤을 때 오래된 상태가 최신 상태를 덮어쓰지 않도록 기존 Entity의 버전 매핑을 복구합니다.

- [ ] 기존 `WorldTrialSite`의 버전 필드를 JPA가 관리하도록 구성합니다. 버전 값을 직접 증가시키거나 비관적 락·전역 동기화로 대체하지 않습니다.
- [ ] **테스트 확인:** `OptimisticLockTest.java`의 주석을 해제한 뒤 실행합니다.
- [ ] **확인:** 같은 버전을 읽은 두 저장 중 하나만 커밋되고, 충돌한 트랜잭션의 다른 행 변경도 함께 롤백되어야 합니다. 서로 다른 월드의 독립된 저장은 모두 성공해야 합니다.

### Lv 17. 커서 페이지 조회 `도전`

구현 위치: `chat/service/ChatHistoryService`

채팅이 계속 추가되는 동안 과거 내역을 여러 페이지로 읽으려고 합니다. 조회 결과를 바탕으로 다음 요청에 사용할 커서를 반환하세요.

**API 명세 → 과거 채팅 커서 조회, `GET /worlds/{worldId}/chats/history`**

> 커서에 대해 공부하시고 문제를 진행해주세요.

- [ ] 생성 시각 내림차순이며 시각이 같으면 ID 내림차순인 조회 결과에서 `last`를 결정합니다. 다음 페이지가 있으면 **실제로 반환하는 `items`의 마지막 항목**을 선택하고, 없으면 `null`을 사용합니다. 추가로 조회한 한 건을 다음 커서로 사용하면 안 됩니다.
- [ ] 첫 요청은 커서 없이 호출합니다. 다음 페이지는 응답의 `nextCreatedAt`과 `nextId`를 각각 `beforeCreatedAt`, `beforeId`로 함께 전달해 조회합니다.
- [ ] **테스트 확인:** `ChatHistoryTest.java`의 주석을 해제한 뒤 실행합니다.
- [ ] **확인:** 제공 테스트에서 다음 커서가 반환한 목록의 마지막 항목인지 확인합니다. 마지막 페이지와 빈 결과에서는 다음 커서가 `null`이어야 합니다.

### Lv 18. Redis 최근 채팅 캐시 `도전`

구현 위치: `chat/service/RecentChatCache`

`read()`, `write()`, `invalidate()`에서 최근 채팅 캐시를 조회하고 저장하며 삭제하는 기능을 구현하세요.

- [ ] `read()`에서 `key(worldId, limit)`에 저장된 JSON 문자열을 조회해 `json`에 대입합니다.
- [ ] `write()`에서 제공된 `json`을 `key(worldId, limit)`에 저장하고 5초의 TTL을 설정합니다. 빈 목록도 저장합니다.
- [ ] `invalidate()`에서 제공된 `keys` 목록에 해당하는 Redis 데이터를 삭제합니다.
- [ ] **테스트 확인:** Docker를 실행하고 `RecentChatCacheTest.java`의 주석을 해제한 뒤 실행합니다.
- [ ] **확인:** 저장한 채팅과 빈 목록을 다시 읽을 수 있고, 키의 TTL이 5초 이내인지 확인합니다. 한 월드의 캐시를 삭제해도 다른 월드의 캐시는 남아 있어야 합니다.

### Lv 19. Redis Lua로 채팅 전송 횟수 제한 `도전`

구현 위치: `chat/service/ChatRateLimitService`

조회한 횟수가 제한보다 작은지 확인한 뒤 별도 명령으로 횟수를 증가시키면, 동시에 들어온 요청들이 모두 통과할 수 있습니다. `allow()`에서 횟수 확인과 증가를 원자적으로 처리하세요.

- `INCR`: 요청 횟수를 1 증가시킵니다.
- `EXPIRE`: 키의 만료 시간을 초 단위로 설정합니다.
- `tonumber(value or '0')`: 키가 없으면 횟수를 0으로 처리합니다.

- [ ] 제공된 테스트를 실행해 동시에 들어온 요청이 제한을 초과해 통과하는 실패를 확인합니다. 플레이어별로 첫 허용 요청부터 10초 동안 새 채팅을 최대 5건 허용하도록 개선합니다. 월드가 달라도 같은 플레이어는 제한을 공유하며 재접속해도 초기화되지 않습니다. Redis 키는 서버별로 나누지 않습니다. 형식 검증을 통과한 채팅 요청을 세며, 요청 식별자나 재전송 중복 판정은 구현하지 않습니다.
- [ ] 횟수 확인, 증가와 최초 만료 설정을 Lua Script로 처리합니다. 후속 요청이 만료 시간을 계속 연장하지 않도록 합니다. 제한된 채팅은 저장하거나 방송하지 않고 제공된 제한 안내로 연결합니다. `allow(playerId)`는 허용 여부를 `boolean`으로 반환합니다.
- [ ] **테스트 확인:** Docker를 실행하고 `ChatRateLimitTest.java`의 주석을 해제한 뒤 실행합니다.
- [ ] **확인:** 제공 테스트에서 동시 요청을 5건까지만 허용하는지, 후속 요청이 제한 시간을 연장하지 않는지 확인합니다. 만료 후에는 다시 허용하고, 다른 플레이어의 한도에는 영향을 주지 않아야 합니다.
