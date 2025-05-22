## 4. 三值邏輯與重複/例外資料查核
### 題目 4-1
**查出 `entry_time `為 NULL 的報名紀錄，並顯示會員與課程名稱**
```
SELECT
  r.registration_id,
  m.member_id,
  m.name AS member_name,
  c.course_id,
  c.name AS course_name
FROM Registrations r
JOIN Members m ON r.member_id = m.member_id
JOIN CourseSchedules cs ON r.course_schedule_id = cs.course_schedule_id
JOIN Courses c ON cs.course_id = c.course_id
WHERE r.entry_time IS NULL;
```
![image](https://hackmd.io/_uploads/Bk3-D4h-xx.png)

### 題目 4-2
**檢查同一會員在同一時段報名多次的情況**
```
SELECT
  r.member_id,
  m.name AS member_name,
  r.course_schedule_id,
  COUNT(*) AS registration_count
FROM Registrations r
JOIN Members m ON r.member_id = m.member_id
GROUP BY r.member_id, r.course_schedule_id, m.name
HAVING COUNT(*) > 1;
```
![截圖 2025-05-22 下午1.33.20](https://hackmd.io/_uploads/Syp7PNhZle.png)

