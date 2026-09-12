# STR-1195 — миграция: назначение сотрудника на соревнование

Таблица по аналогии с `events_bids_fighters` / `events_bids_judges`. Полномочия — отдельной колонкой на каждое право (как `approved`, `approve_weight` у бойцов), а не строкой флагов.

```sql
--liquibase formatted sql

--changeset strikerstat:2026_09_20_000001_create_events_bids_staff
--comment Назначение сотрудника на соревнование и его полномочия в рамках этого соревнования
--preconditions onFail:HALT
--precondition-sql-check expectedResult:0 SELECT COUNT(*) FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA = DATABASE() AND TABLE_NAME = 'events_bids_staff'
CREATE TABLE events_bids_staff
(
    id                    BIGINT       NOT NULL AUTO_INCREMENT,
    event_id              INT          NOT NULL,
    staff_id              INT          NOT NULL,

    -- полномочия на этом соревновании
    can_weigh_in          TINYINT(1)   NOT NULL DEFAULT 0,
    can_interview         TINYINT(1)   NOT NULL DEFAULT 0,
    can_sortition         TINYINT(1)   NOT NULL DEFAULT 0,
    can_edit_fighter      TINYINT(1)   NOT NULL DEFAULT 0,
    can_add_fighter       TINYINT(1)   NOT NULL DEFAULT 0,
    can_set_group         TINYINT(1)   NOT NULL DEFAULT 0,
    can_set_result        TINYINT(1)   NOT NULL DEFAULT 0,
    can_delete_battle     TINYINT(1)   NOT NULL DEFAULT 0,
    can_upload_media      TINYINT(1)   NOT NULL DEFAULT 0,
    can_broadcast         TINYINT(1)   NOT NULL DEFAULT 0,
    can_comment           TINYINT(1)   NOT NULL DEFAULT 0,
    can_accept_payment    TINYINT(1)   NOT NULL DEFAULT 0,
    can_export            TINYINT(1)   NOT NULL DEFAULT 0,
    can_view_payment_sums TINYINT(1)   NOT NULL DEFAULT 0,
    can_edit_fighter_card TINYINT(1)   NOT NULL DEFAULT 0,

    -- что скрыто в таблице
    hide_payments         TINYINT(1)   NOT NULL DEFAULT 0,
    hide_phones           TINYINT(1)   NOT NULL DEFAULT 0,
    hide_comments         TINYINT(1)   NOT NULL DEFAULT 0,

    -- снятие с соревнования без удаления истории
    active                TINYINT(1)   NOT NULL DEFAULT 1,
    revoked_at            DATETIME(6)  NULL,

    created_at            DATETIME(6)  NOT NULL,
    created_by            VARCHAR(255) NULL,
    modified_at           DATETIME(6)  NULL,
    modified_by           VARCHAR(255) NULL,

    CONSTRAINT pk_events_bids_staff PRIMARY KEY (id),
    CONSTRAINT uq_events_bids_staff_event_staff UNIQUE (event_id, staff_id),
    CONSTRAINT fk_events_bids_staff_event FOREIGN KEY (event_id) REFERENCES events (event_id),
    CONSTRAINT fk_events_bids_staff_staff FOREIGN KEY (staff_id) REFERENCES staff (id),
    INDEX idx_events_bids_staff_staff (staff_id),
    INDEX idx_events_bids_staff_event_active (event_id, active)
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4 COLLATE = utf8mb4_unicode_ci;

--rollback DROP TABLE events_bids_staff;
```

## Заметки

- **Имена FK уникальны в пределах схемы.** В первом варианте был скопирован `events_bids_fighters_fighter_id_foreign` — такой constraint уже существует, `CREATE TABLE` упал бы с «Duplicate foreign key constraint name».
- **`AUTO_INCREMENT=84553`** из дампа чужой таблицы убран.
- **`UNIQUE (event_id, staff_id)`** — один сотрудник не может быть назначен на событие дважды.
- **`active` + `revoked_at`** вместо `DELETE`: на назначение ссылаются записи об оплате, удаление строки порвало бы FK и стёрло автора платежа.
- **`can_edit_fighter_card`** — это STR-1300 (карточка спортсмена на ОР). Колонка заводится сразу, хотя чекбокс пока неактивен, чтобы потом не делать миграцию ради одной галочки.
- **Те же колонки нужны в `staff`** как шаблон по умолчанию из окна создания сотрудника. Названия один в один — тогда массовое назначение пишется как `INSERT ... SELECT`, а права **копируются**, а не читаются по ссылке: правка шаблона не должна менять задним числом уже выданные полномочия.

```sql
INSERT INTO events_bids_staff (event_id, staff_id, can_weigh_in, ..., created_at)
SELECT e.event_id, s.id, s.can_weigh_in, ..., NOW(6)
FROM staff s CROSS JOIN events e
WHERE s.id = ? AND e.event_id IN (...);
```

- **Платежи ссылаются на назначение, а не на сотрудника:** `events_staff_payments.events_bids_staff_id` → «кто принял оплату на этом турнире» берётся одним join'ом, лимит 5 проверяется через `COUNT(*) WHERE bid_id = ?`.
- **На бэке** завести enum с функцией-геттером, чтобы аспект не разрастался ветками:

```java
public enum StaffPermission {
    WEIGH_IN(EventBidStaff::isCanWeighIn),
    SORTITION(EventBidStaff::isCanSortition),
    ACCEPT_PAYMENT(EventBidStaff::isCanAcceptPayment),
    // ...
    ;
    private final Predicate<EventBidStaff> check;
}
```
