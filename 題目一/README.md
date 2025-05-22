## 1. 查詢與效能運算練習
### 題目 1-1
**列出「過去 6 個月未曾進場過的會員」的 member_id 與 name。**
1. `NOT IN`：
```
SELECT member_id, name
FROM Members
WHERE member_id NOT IN (
    SELECT DISTINCT member_id
    FROM Registrations
    WHERE entry_time IS NOT NULL
      AND entry_time >= DATE_SUB(CURDATE(), INTERVAL 6 MONTH)
);
```
2. `NOT EXISTS`：
```
SELECT member_id, name
FROM Members m
WHERE NOT EXISTS (
  SELECT 1
  FROM Registrations r
  WHERE r.member_id = m.member_id
    AND r.entry_time IS NOT NULL
    AND r.entry_time >= DATE_SUB(CURDATE(), INTERVAL 6 MONTH)
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
      AND entry_time >= DATE_SUB(CURDATE(), INTERVAL 6 MONTH)
) r ON m.member_id = r.member_id
WHERE r.member_id IS NULL;
```
![截圖 2025-05-22 下午1.03.51](https://hackmd.io/_uploads/rkaHeV2Wxe.png)
### 題目 1-2
**列出同時報名過兩個指定時段（假設 `course_schedule_id = 'A'`、`B'`）的會員**
* A、Ｂ換成course_schedule_id的其中兩個
1. `GROUP BY ... HAVING COUNT`：
```
SELECT member_id 
FROM Registrations 
WHERE course_schedule_id IN ('A', 'B') 
GROUP BY member_id 
HAVING COUNT(DISTINCT course_schedule_id) = 2;
```
![截圖 2025-05-22 下午1.07.45](https://hackmd.io/_uploads/rJ1NZE3Zll.png)

2. `EXISTS 子查詢`
```
SELECT m.member_id, m.name
FROM Members m
WHERE EXISTS (
    SELECT 1 FROM Registrations r1 WHERE r1.member_id = m.member_id AND r1.course_schedule_id = 'A'
)
AND EXISTS (
    SELECT 1 FROM Registrations r2 WHERE r2.member_id = m.member_id AND r2.course_schedule_id = 'B'
);
```
![截圖 2025-05-22 下午1.08.10](https://hackmd.io/_uploads/BkwHZVhZgx.png)

3. `JOIN` :
```
SELECT m.member_id, m.name
FROM Members m
JOIN Registrations rA ON m.member_id = rA.member_id AND rA.course_schedule_id = 'A'
JOIN Registrations rB ON m.member_id = rB.member_id AND rB.course_schedule_id = 'B';
```
![截圖 2025-05-22 下午1.09.17](https://hackmd.io/_uploads/Hy9FbEhWle.png)