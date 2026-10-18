**token_setproc.c.patch**: работа без модуля ядра access_token_id

Без `/dev/access_token_id` функции не возвращают ошибку, а хранят токен сами:
  - в памяти процесса (g_selfTokenId, g_firstTokenId);
  - в `/data/service/el0/access_token/bypid/<pid>` — оттуда токен отправителя
    читает binder_invoker (нужен парный патч в foundation/communication/ipc);
  - в `/data/service/el0/access_token/byname/<имя>` — токены служб, файлы
    раскладывает `token_init`.
    
Применить (из корня дерева OpenHarmony):
  * `P=/mnt/ohos/OpenHarmony7/oneir_patches/kernel_depending/token_setproc.c.patch`
  * `git -C base/security/access_token apply --check "$P"`
  * `git -C base/security/access_token apply "$P"`

Откатить:
  * `git -C base/security/access_token restore -- interfaces/innerkits/token_setproc/src/token_setproc.c`

Проверить:
  * `git -C base/security/access_token status --short`

---

**binder_invoker.cpp.patch**: токен отправителя без модуля ядра access_token_id
Парный патч к token_setproc-no-tokenid.patch (тот пишет `bypid/<pid>`).
Если ядро не передаёт токены в binder (нет `SenderInfo` и `AccessToken`):
  - LookupTokenByPid(pid): bypid/<pid>, иначе `byname/<имя из /proc/<pid>/cmdline>`;
  - Transaction: токен вызывающего ищется по `callerPid_;` если не найден
    (например, oneway-вызов, pid = 0) — берётся токен самого получателя;
    firstTokenID_ = callerTokenID_;
  - GetSelfTokenID / GetSelfFirstCallerTokenID: вместо uid — свой токен
    из bypid/byname, uid только если записи нет.
    
Ограничение: при неизвестном отправителе вызов проверяется с правами
получателя, а pid может достаться новому процессу — чистите `bypid/`
перед запуском стенда.

Применить (из корня дерева OpenHarmony):
  * `P=/mnt/ohos/OpenHarmony7/oneir_patches/kernel_depending/binder_invoker.cpp.patch`
  * `git -C foundation/communication/ipc apply --check "$P"`
  * `git -C foundation/communication/ipc apply "$P"`
Откатить:
  * `git -C foundation/communication/ipc restore -- ipc/native/src/core/invoker/source/binder_invoker.cpp`
