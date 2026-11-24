# Muscle Data Develpment from sql
## What is the use case.

Have a database in sql that will store data and have a store procedure that makes use of the existing tables


## Server name?

DESKTOP-99DDSKS\SQLEXPRESS
## store procedure
```sql
USE [Muscle]
GO
/****** Object:  StoredProcedure [Muscle].[MUSCLE_DATE_SP_USE]    Script Date: 11/24/2026 11:27:04 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [Muscle].[MUSCLE_DATE_SP_USE]
AS
BEGIN
/*
--RENAME COLUMN
  SELECT * FROM MUSCLE.Loans
  SELECT * FROM [Muscle].[Muscle].[mGR]
  EXEC sp_rename '[Muscle].[Muscle].[Loans].[DATE_RECEIVED_LOAN]',  'DATE_RECEIVED', 'COLUMN';
   EXEC sp_rename '[Muscle].[Muscle].[Loans].[I_DATE_LOAN]',  'I_DATE', 'COLUMN';
   EXEC sp_rename '[Muscle].[Muscle].[Loans].[TIER_LOAN]',  'TIER', 'COLUMN';
   EXEC SP_RENAME '[Muscle].[Muscle].[mGR].[AMOUNT_MGR]','AMOUNT','COLUMN';
   EXEC SP_RENAME '[MUSCLE].[MUSCLE].RECEIVER.[MONTH_RECEIVER1]','MONTH','COLUMN';
   EXEC SP_RENAME '[MUSCLE].[MUSCLE].RECEIVER.[ID_RECEIVER1]','ID','COLUMN';
   EXEC SP_RENAME '[MUSCLE].[MUSCLE].RECEIVER.[YEAR_RECEIVER1]','YEAR','COLUMN';
   EXEC SP_RENAME '[MUSCLE].[MUSCLE].MUSCLE_EMAILS_LIST.ID_MUSCLE_EMAILS_LIST','ID','COLUMN';
   */

    update muscle.savings_alteryx
   set [MEMBER SENT]='BRITON AMBALE'
   WHERE ID=22

   update muscle.MGR_alteryx
   set [MEMBER SENT]='BRITON AMBALE'
   WHERE ID=22
   --INSERT INTO SAVINGS
insert into muscle.savings
--select  *,0 as max_mgr_flag from muscle.savings_alteryx
 
   select  * from Muscle.savings_raw_cleaned
   where CONCAT(month,year) not in 
   (
   select distinct CONCAT(month,year) from Muscle.savings
   )
---select  top 1 * from muscle.savings
--INSERT INTO MGR
INSERT INTO MUSCLE.mgr
---select *,0 as max_mgr_flag from muscle.select  * from Muscle.savings_raw_cleaned

    select  * from Muscle.mgr_raw_cleaned
   where CONCAT(month,year) not in 
   (
   select distinct CONCAT(month,year) from MUSCLE.mgr
   )


--INSERT INTO WELFARE
INSERT INTO MUSCLE.WELFARE
 select  * from Muscle.welfare_raw_cleaned
   where CONCAT(Updated_date,Month) not in 
   (
   select CONCAT(Updated_date,Month) from MUSCLE.welfare
   )

   --INSERT INTO email
INSERT INTO muscle.muscle_Emails_List
 select  * from  muscle.emails_raw_file
   where CONCAT(full_name,year) not in 
   (
   select CONCAT(Full_name,year) from muscle.muscle_Emails_List
   )

--SELECT * FROM muscle.savings WHERE max_mgr_flag=1
 update muscle.MGR
   set [MEMBER_SENT]='BRITON AMBALE'
   WHERE ID=21
   

   DROP TABLE IF EXISTS MUSCLE.month_group__by_year
   SELECT YEAR,
   CASE
     WHEN "GROUP"= 'disbursment' THEN 'Amount disbursed By Year'
	 WHEN "GROUP"= 'SV' THEN 'Amount saved By Year'
	 ELSE "GROUP" END AS "GROUP"
   ,SUM(M_AMOUNT) AMOUNT 
   INTO MUSCLE.month_group__by_year
   FROM Muscle.savings
   --WHERE M_AMOUNT>0
   GROUP BY YEAR,"GROUP"



   SELECT * FROM MUSCLE.month_group__by_year
   --updating sent column
   update muscle.savings
set SENT=1
where M_AMOUNT>0
   --UPDATING WELFARE
   update muscle.Welfare
set reason_for_cont='EXPENSE'
WHERE AMOUNT_KSH<=0

---CREATE TABLE MUSCLE.SAVINGS_MONTH(MEMBER_SENT VARCHAR(100),MONTH VARCHAR(100),YEAR INT,SAVINGS INT);
DROP TABLE IF EXISTS MUSCLE.SAVINGS_MONTH
CREATE TABLE MUSCLE.SAVINGS_MONTH(MEMBER_SENT VARCHAR(100),MONTH VARCHAR(100),YEAR INT,SAVINGS INT,id int,your_email varchar(100));
INSERT  INTO Muscle.SAVINGS_MONTH (MEMBER_SENT,MONTH,YEAR,SAVINGS,id,your_Email)
SELECT  distinct MEMBER_SENT,MONTH,sv.YEAR,SUM(M_AMOUNT) AS SAVINGS,sv.id,li.Your_Email
FROM Muscle.savings sv
left join (select distinct id,Your_Email from muscle.muscle_Emails_List) li on li.ID=sv.id
WHERE [group]='sv' --and sv.year=year(getdate())
--and sv.id=5
GROUP BY MEMBER_SENT
,MONTH,sv.YEAR,sv.id,li.Your_Email

SELECT * FROM MUSCLE.SAVINGS_MONTH
where year=2026
--update  muscle.mgr
--set updated_date=CURRENT_TIMESTAMP


--update loan interest
UPDATE muscle.savings 
SET LOAN_INTERESTET =  (
    SELECT SUM(b.INTEREST_AMOUNT_10)
    FROM muscle.Loans b
    WHERE b.ID = muscle.savings.id and b.MONTH=muscle.savings.MONTH AND B.year=muscle.savings.YEAR
)
WHERE EXISTS (
    SELECT 1
    FROM muscle.Loans b
    WHERE b.id = muscle.savings.id and b.MONTH=muscle.savings.MONTH AND B.year=muscle.savings.YEAR
);
--select YEAR,MEMBER_SENT,sum(LOAN_INTERESTET) AS INTEREST from muscle.savings
--where LOAN_INTERESTET is not null and YEAR=2024
--GROUP BY YEAR,MEMBER_SENT
--ORDER BY YEAR

--update loan interest
update  muscle.Loans
set INTEREST_AMOUNT_10=0.1*AMOUNT_KSH

--UPDATE TARGET AMOUNT
UPDATE P
  SET [TARGET AMOUNT]=L.[TARGET AMOUNT]
  FROM
  muscle.savings P 
  LEFT JOIN [Muscle].[Muscle].[muscle_Emails_List] L
  ON P.id=L.ID
  AND P.[YEAR]=L.[Year]


  ---update mgrflag
  update muscle.savings
set max_mgr_flag= case when I_DATE IN (SELECT MAX(I_DATE) FROM MUSCLE.savings) then 1
else 0 end

 update muscle.mgr
set max_mgr_flag= case when I_DATE IN (SELECT MAX(I_DATE) FROM MUSCLE.mgr) then 1
else 0 end

update muscle.WELFARE
set max_mgr_flag= case when first_of_month IN (SELECT MAX(first_of_month) FROM MUSCLE.welfare) then 1
else 0 end

/*
UPDATE MUSCLE.loans
SET AMOUNT_PAID=1650,
DATE_PAID='2024-11-11',
RECEIPT_MPESA_BANK_OF_LOANPAY='SKB0FHJPPE Confirmed.You have received Ksh2,475.00 from EUNICE  ETEMESI 0113111928 on 11/11/24 at 4:46 PM  New M-PESA balance is Ksh3,572.06. Sign up for Lipa Na M-PESA Till online https://m-pesaforbusiness.co.ke'
WHERE RECEIPT_MPESA_BANK_OF_LOAN_RECEIVED='SJL9TT5XLR Confirmed. Ksh1,500.00 sent to EUNICE ETEMESI 0113111928 on 21/10/24 at 9:36 AM. New M-PESA balance is Ksh2,411.05. Transaction cost, Ksh23.00. Amount you can transact within the day is 498,380.00. Thank you for being our valued customer, we appreciate you.'
*/

/*
update date format

update muscle.savings
set date=SUBSTRING(CONVERT(VARCHAR, [date], 120), 1, 4) + '-' +  -- Year part
          SUBSTRING(CONVERT(VARCHAR, [date], 120), 9, 2) + '-' +  -- Month part
          SUBSTRING(CONVERT(VARCHAR, [date], 120), 6, 2)
where MONTH='july'
and year=2024 and date!=DATE_RECEIVED
and date='2024-09-07'


--update emails list
INSERT INTO muscle.muscle_Emails_List (FULL_NAME,ID,NATIONA_ID,YOUR_EMAIL,YEAR,[target Amount],
[Do u agree with the MGR strategy shared during the meeting?],[If No, why?])
SELECT MEMBER AS FULL_NAME,
ID,
ID AS NATIONAL_ID,
CONCAT('MCO',ID) AS YOUR_EMAIL,
YEAR,
0 AS [TARGET AMOUNT],
'Yes' as [Do u agree with the MGR strategy shared during the meeting?],
'N/A' as [If No, why?]
FROM MUSCLE.RECEIVER
WHERE ID NOT IN 
(select DISTINCT ID from muscle.muscle_Emails_List where year=2025)
and year=2025
*/
--Update date
/*
update muscle.mgr
set updated_date=CURRENT_TIMESTAMP
where YEAR=year(getdate())
and I_DATE=(select max(i_Date) from muscle.mgr)
*/
END;


```
