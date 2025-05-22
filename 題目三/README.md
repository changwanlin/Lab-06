## 3. 聚合與索引優化練習
### 題目 3-1
**列出每位教練、其課程、課次總數，及平均每場報名人數**
```
SELECT
  s.staff_id AS coach_id,
  s.name AS coach_name,
  c.course_id,
  c.name AS course_name,
  COUNT(DISTINCT cs.course_schedule_id) AS total_sessions,
  ROUND(
    IFNULL(COUNT(r.registration_id), 0) / NULLIF(COUNT(DISTINCT cs.course_schedule_id), 0), 2
  ) AS avg_registration_per_session
FROM StaffAccounts s
JOIN Courses c ON c.coach_id = s.staff_id
JOIN CourseSchedules cs ON cs.course_id = c.course_id
LEFT JOIN Registrations r ON r.course_schedule_id = cs.course_schedule_id
WHERE s.role = 'COACH'
GROUP BY s.staff_id, s.name, c.course_id, c.name
ORDER BY s.staff_id, c.course_id;
```
![image](https://hackmd.io/_uploads/ry2q8VnWlx.png)

### 題目 3-2
**找出三個月內報到次數最多的 10 名會員（member_id, name, 出席次數）**
```
SELECT
  m.member_id,
  m.name,
  COUNT(r.entry_time) AS checkin_times
FROM Members m
JOIN Registrations r ON r.member_id = m.member_id
WHERE r.entry_time IS NOT NULL
  AND r.entry_time >= DATE_SUB(CURDATE(), INTERVAL 3 MONTH)
GROUP BY m.member_id, m.name
ORDER BY checkin_times DESC
LIMIT 10;
```
![截圖 2025-05-22 下午1.31.20](https://hackmd.io/_uploads/HJPhIN2ble.png)