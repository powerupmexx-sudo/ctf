# Padel (web) — решение

## Уязвимость
В `handleLeaveRoom` сервер берет `leavingId` из пользовательского сообщения, а не фиксирует его текущим `playerId` соединения:

- `const { playerId: leavingId } = { playerId, ...msg };`

Это дает возможность отправить `c_leave_room` с произвольным `playerId` и «выгнать» любого игрока/бота из текущей комнаты.

## Почему это дает флаг
Для official-комнат:

1. Изначально хост — бот (добавляется первым).
2. История чата с «слухами» и строкой с флагом хранится в `chatMessages`.
3. При смене хоста на реального игрока (`removePlayer`) вызывается `_replayChatHistoryTo(newHost.ws)`.

Значит, нужно стать хостом в official-комнате: удалить всех ботов через подмену `playerId` в `c_leave_room`.

## Эксплуатация
Скрипт `exploit.py` делает:

1. регистрирует пользователя;
2. открывает websocket с `padel_sid`;
3. запрашивает `s_official_list`, заходит в official room;
4. получает `s_room_state`, собирает `id` ботов;
5. отправляет для каждого бота `{"type":"c_leave_room","playerId":"<bot_id>"}`;
6. после получения хоста читает реплей чата и вытаскивает `alfa{...}`.

## Запуск
```bash
pip install requests websocket-client
python3 solution/exploit.py
```

Если сеть недоступна из вашей среды, запустите скрипт локально на машине с доступом к `padel-extkzrin.alfactf.ru`.
