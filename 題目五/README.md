## 5. 多表關聯與綜合查詢效能
### 題目 5-1
**列出每位教練「本月」課程所有時段的平均出席人數**
```
SELECT
  s.staff_id AS coach_id,
  s.name AS coach_name,
  ROUND(
    IFNULL(SUM(entry_count), 0) / NULLIF(COUNT(cs.course_schedule_id), 0),
    2
  ) AS avg_attendance_per_session
FROM StaffAccounts s
JOIN Courses c ON c.coach_id = s.staff_id
JOIN CourseSchedules cs ON cs.course_id = c.course_id
LEFT JOIN (
  SELECT
    r.course_schedule_id,
    COUNT(*) AS entry_count
  FROM Registrations r
  WHERE r.entry_time IS NOT NULL
    AND r.entry_time >= DATE_FORMAT(CURDATE(), '%Y-%m-01')
    AND r.entry_time < DATE_FORMAT(DATE_ADD(CURDATE(), INTERVAL 1 MONTH), '%Y-%m-01')
  GROUP BY r.course_schedule_id
) rc ON rc.course_schedule_id = cs.course_schedule_id
WHERE s.role = 'COACH'
  AND cs.start_time >= DATE_FORMAT(CURDATE(), '%Y-%m-01')
  AND cs.start_time < DATE_FORMAT(DATE_ADD(CURDATE(), INTERVAL 1 MONTH), '%Y-%m-01')
GROUP BY s.staff_id, s.name
ORDER BY avg_attendance_per_session DESC;
```
![截圖 2025-05-22 下午1.33.57](https://hackmd.io/_uploads/BJzUP4nbgg.png)

### 題目 5-2
**列出一年內「從未缺席任何一場已報名時段」的學員名單（全勤會員）**
```
SELECT m.member_id, m.name
FROM Members m
WHERE EXISTS (
    SELECT 1
    FROM Registrations r
    WHERE r.member_id = m.member_id
      AND r.register_time >= DATE_SUB(CURDATE(), INTERVAL 1 YEAR)
)
AND NOT EXISTS (
    SELECT 1
    FROM Registrations r2
    WHERE r2.member_id = m.member_id
      AND r2.register_time >= DATE_SUB(CURDATE(), INTERVAL 1 YEAR)
      AND r2.entry_time IS NULL
);
```
![截圖 2025-05-22 下午1.34.26](https://hackmd.io/_uploads/Byx_vVnZle.png)
