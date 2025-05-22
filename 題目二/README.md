## 2. 子查詢與 EXISTS 效能比較
### 題目 2-1
**找出「本月內沒有任何進場記錄」的會員**
1. `NOT EXISTS`：
```
SELECT m.member_id, m.name
FROM Members m
WHERE NOT EXISTS (
    SELECT 1
    FROM Registrations r
    WHERE r.member_id = m.member_id
      AND r.entry_time IS NOT NULL
      AND r.entry_time >= DATE_FORMAT(CURDATE(), '%Y-%m-01')
      AND r.entry_time < DATE_FORMAT(DATE_ADD(CURDATE(), INTERVAL 1 MONTH), '%Y-%m-01')
);
```

2. `NOT IN`：
```
SELECT m.member_id, m.name
FROM Members m
WHERE m.member_id NOT IN (
    SELECT DISTINCT r.member_id
    FROM Registrations r
    WHERE r.entry_time IS NOT NULL
      AND r.entry_time >= DATE_FORMAT(CURDATE(), '%Y-%m-01')
      AND r.entry_time < DATE_FORMAT(DATE_ADD(CURDATE(), INTERVAL 1 MONTH), '%Y-%m-01')
);
```

3. `LEFT JOIN ... IS NULL`：
```
SELECT m.member_id, m.name
FROM Members m
LEFT JOIN (
    SELECT DISTINCT member_id
    FROM Registrations
    WHERE entry_time IS NOT NULL
      AND entry_time >= DATE_FORMAT(CURDATE(), '%Y-%m-01')
      AND entry_time < DATE_FORMAT(DATE_ADD(CURDATE(), INTERVAL 1 MONTH), '%Y-%m-01')
) r ON m.member_id = r.member_id
WHERE r.member_id IS NULL;
```
![截圖 2025-05-22 下午1.09.40](https://hackmd.io/_uploads/H1Msb4hbxe.png)
### 題目 2-2
**列出「至少曾參加自己擔任教練課程」的教練清單**
`EXISTS子查詢`：
```
SELECT s.staff_id, s.name
FROM StaffAccounts s
WHERE s.role = 'COACH'
  AND EXISTS (
        SELECT 1
        FROM Courses c
        JOIN CourseSchedules cs ON c.course_id = cs.course_id
        JOIN Registrations r ON cs.course_schedule_id = r.course_schedule_id
        JOIN Members m ON r.member_id = m.member_id
        WHERE c.coach_id = s.staff_id
          AND m.member_code = s.staff_code
      );
```

![截圖 2025-05-22 下午1.11.29](https://hackmd.io/_uploads/r1j-ME3-lx.png)
