**token_setproc**: работа без модуля ядра access_token_id

Без `/dev/access_token_id` функции не возвращают ошибку, а хранят токен сами:
  - в памяти процесса (g_selfTokenId, g_firstTokenId);
  - в `/data/service/el0/access_token/bypid/<pid>` — оттуда токен отправителя
    читает binder_invoker (нужен парный патч в foundation/communication/ipc);
  - в `/data/service/el0/access_token/byname/<имя>` — токены служб, файлы
    раскладывает `token_init`.
    
Применить (из корня дерева OpenHarmony):
  * `P=/mnt/ohos/OpenHarmony7/oneir_patches/kernel_depending/token_setproc.c.patch`
  * `git -C base/security/access_token apply --check -p4 "$P"`
  * `git -C base/security/access_token apply -p4 "$P"`

Откатить:
  * `git -C base/security/access_token restore -- interfaces/innerkits/token_setproc/src/token_setproc.c`
