# operational-alert-sla-sql-project

SQL project analyzing operational alerts and SLA performance

[Operational Alert & SLA Performance Analysis using SQL.sql](https://github.com/user-attachments/files/24523594/Operational.Alert.SLA.Performance.Analysis.using.SQL.sql)

-- Project 1 : Operational / SLA / Alert Performance Analysis.
DROP TABLE IF EXISTS alerts;
DROP TABLE IF EXISTS sla_targets;

CREATE TABLE alerts (
    alert_id INT,
    alert_type VARCHAR(50),
    priority VARCHAR(20),
    created_time DATETIME,
    acknowledged_time DATETIME,
    status VARCHAR(20)
);

CREATE TABLE sla_targets (
    priority VARCHAR(20),
    sla_minutes INT
);

INSERT INTO sla_targets VALUES
('High', 60),
('Medium', 90),
('Low', 120);

INSERT INTO alerts VALUES
(1,'Temperature Alert','High','2025-01-01 10:00:00','2025-01-01 10:40:00','Acknowledged'),
(2,'Power Failure','High','2025-01-01 11:00:00','2025-01-01 12:30:00','Acknowledged'),
(3,'Sensor Offline','Medium','2025-01-01 09:30:00','2025-01-01 11:00:00','Acknowledged'),
(4,'Door Open','Low','2025-01-01 08:00:00','2025-01-01 10:30:00','Acknowledged'),
(5,'Temperature Alert','High','2025-01-02 14:00:00',NULL,'Missed'),
(6,'Sensor Offline','Medium','2025-01-02 15:00:00','2025-01-02 16:50:00','Acknowledged'),
(7,'Power Failure','High','2025-01-03 10:00:00','2025-01-03 10:50:00','Acknowledged'),
(8,'Door Open','Low','2025-01-03 11:00:00','2025-01-03 14:00:00','Acknowledged'),
(9,'Temperature Alert','Medium','2025-01-04 09:00:00',NULL,'Missed'),
(10,'Sensor Offline','Low','2025-01-04 12:00:00','2025-01-04 14:30:00','Acknowledged');

-- 1. Total alerts by priority
-- Script :
select priority, count(*) as total_alerts
from alerts
group by priority
order by total_alerts DESC;

-- 2. Average response time (minutes) by priority
-- Script:
select a.priority, 
    round(avg(timestampdiff(minute, a.created_time, a.acknowledged_time)), 1) AS avg_response_times 
    from alerts a
where a.acknowledged_time is not null
group by a.priority;

-- 3.Alerts that breached SLA
-- Script : 
SELECT a.alert_id,
       a.alert_type,
       a.priority,
       TIMESTAMPDIFF(MINUTE, a.created_time, a.acknowledged_time) AS response_minutes,
       s.sla_minutes
FROM alerts a
JOIN sla_targets s
  ON a.priority = s.priority
WHERE a.acknowledged_time IS NOT NULL
  AND TIMESTAMPDIFF(MINUTE, a.created_time, a.acknowledged_time) > s.sla_minutes;
  
  -- 4. SLA breach percentage by priority
-- Script : 
SELECT a.priority,
       ROUND(
         100.0 * SUM(
           CASE
             WHEN a.acknowledged_time IS NOT NULL
              AND TIMESTAMPDIFF(MINUTE, a.created_time, a.acknowledged_time) > s.sla_minutes
             THEN 1 ELSE 0
           END
         ) / COUNT(*),
       1) AS breach_percentage
FROM alerts a
JOIN sla_targets s
  ON a.priority = s.priority
GROUP BY a.priority;

-- 5. Missed vs Acknowledged alerts
-- Script :
SELECT priority, status, COUNT(*) AS alert_count
FROM alerts
GROUP BY priority, status
ORDER BY priority, status;

-- 6. Top alert types causing SLA breaches
-- Script : 
SELECT a.alert_type,
       COUNT(*) AS breach_count
FROM alerts a
JOIN sla_targets s
  ON a.priority = s.priority
WHERE a.acknowledged_time IS NOT NULL
  AND TIMESTAMPDIFF(MINUTE, a.created_time, a.acknowledged_time) > s.sla_minutes
GROUP BY a.alert_type
ORDER BY breach_count DESC;






