>[!question]- запрос на получение количества записей
> Повторения больше одного раза - 
   SELECT fighter_id, event_id, COUNT(*) as duplicate_count FROM events_bids_fighters GROUP BY fighter_id, event_id HAVING COUNT(*) > 1;

>[!question]- как найти выражение а БД
>так

[olympic_events_weight_categories 
    {
        "ring": "A",
        "fightDay": "2025-06-21",
        "timeSlot": 1,
        "fightTime": "12:00"
    },
    {
        "ring": "B",
        "fightDay": "2025-06-21",
        "timeSlot": 1,
        "fightTime": "12:00"
    },
    {
        "ring": "A",
        "fightDay": "2025-06-25",
        "timeSlot": 1,
        "fightTime": "12:00"
    }
]