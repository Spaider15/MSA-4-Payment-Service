# Задание 2. State Machine

Таблица переходов для платежа.

| Исходное состояние | Переходное состояние | Событие |
|---|---|---|
| - | CREATED | Клиент инициировал платеж |
| CREATED | DEBITING | Отправлена команда на списание |
| CREATED | CANCELLED | Клиент отменил платеж (деньги еще не списаны) |
| DEBITING | DEBITED | Деньги списаны |
| DEBITING | FAILED | Ошибка списания (не хватает денег, счет заблокирован) |
| DEBITING | DEBITING | Таймаут / техническая ошибка, повтор |
| DEBITED | FRAUD_CHECKING | Запущена антифрод-проверка |
| DEBITED | REFUNDING | Клиент отменил платеж |
| FRAUD_CHECKING | APPROVED | Антифрод вернул разрешение |
| FRAUD_CHECKING | REFUNDING | Антифрод вернул запрет |
| FRAUD_CHECKING | MANUAL_REVIEW | Антифрод вернул "ручная проверка" |
| FRAUD_CHECKING | APPROVED | Сработал cut-off (сервисы проверок не ответили вовремя) |
| FRAUD_CHECKING | REFUNDING | Клиент отменил платеж |
| MANUAL_REVIEW | APPROVED | Оператор разрешил операцию |
| MANUAL_REVIEW | REFUNDING | Оператор запретил операцию |
| MANUAL_REVIEW | APPROVED | Прошло 20 минут без решения (cut-off) |
| MANUAL_REVIEW | REFUNDING | Клиент отменил платеж (задача оператора отзывается) |
| APPROVED | TRANSFERRING | Отправлена команда перевода контрагенту |
| TRANSFERRING | TRANSFERRING | Таймаут / техническая ошибка, повтор |
| TRANSFERRING | TRANSFERRED | Перевод выполнен |
| TRANSFERRING | REFUNDING | Банк окончательно отклонил перевод |
| TRANSFERRED | COMPLETED | Статус обновлен, проводка в учете, клиент уведомлен |
| REFUNDING | REFUNDING | Ошибка возврата, повтор |
| REFUNDING | DECLINED | Деньги возвращены (причина - запрет антифрода/оператора) |
| REFUNDING | CANCELLED | Деньги возвращены (причина - отмена клиентом) |
| REFUNDING | FAILED | Деньги возвращены (причина - не прошел перевод) |

Финальные состояния: COMPLETED, DECLINED, CANCELLED, FAILED.

Примеры сценариев:
- Успешный: CREATED -> DEBITING -> DEBITED -> FRAUD_CHECKING -> APPROVED -> TRANSFERRING -> TRANSFERRED -> COMPLETED
- Отказ антифрода: ... -> FRAUD_CHECKING -> REFUNDING -> DECLINED (параллельно уходит уведомление в безопасность)
- Ручная проверка: ... -> FRAUD_CHECKING -> MANUAL_REVIEW -> APPROVED / REFUNDING
- Cut-off: ... -> FRAUD_CHECKING (или MANUAL_REVIEW) -> APPROVED -> TRANSFERRING -> ...

После TRANSFERRED отмену клиентом уже не принимаем - это отдельный процесс возврата.
