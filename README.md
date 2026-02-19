# SQL-Payments-Reconciliation-Analytics
** Overview**

This project focuses on financial reconciliation and payment tracking using SQL Server.

It analyzes:

Payment status monitoring

Daily payment trends

Retailer-level total payments

Order outstanding balances

Delivered orders without confirmed payments

Payment automation using stored procedures

All business logic was implemented using T-SQL.

** Business Objectives**

Track payment lifecycle (Pending → Confirmed → Reversed)

Identify delivered but unpaid orders

Calculate outstanding balances per order

Measure retailer total payments vs credit limit

Monitor daily cash inflow trends

Automate payment insertion with stored procedure

** Technical Skills Demonstrated**

CRUD operations (INSERT, UPDATE, DELETE)

INNER JOIN & LEFT JOIN

GROUP BY & aggregation

Conditional aggregation (CASE WHEN)

NULL handling (IS NULL, COALESCE)

Date filtering (GETDATE, DATEADD)

Stored procedure creation (usp_AddPayment)

Financial reconciliation logic

** Key SQL Highlights**
✔ Total Paid vs Outstanding per Order
SELECT 
    O.OrderNumber,
    O.NetAmount,
    COALESCE(SUM(P.Amount), 0) AS TotalPaid,
    O.NetAmount - COALESCE(SUM(P.Amount), 0) AS Outstanding
FROM SalesOrders O
LEFT JOIN Payments P
    ON O.OrderID = P.OrderID
GROUP BY O.OrderNumber, O.NetAmount;

✔ Delivered Orders With No Confirmed Payment
SELECT O.OrderNumber
FROM SalesOrders O
LEFT JOIN Payments P
    ON O.OrderID = P.OrderID
    AND P.Status = 'Confirmed'
WHERE O.Status = 'Delivered'
AND P.PaymentID IS NULL;

✔ Daily Payment Totals (Last 30 Days)
SELECT 
    CAST(PaymentDate AS DATE) AS PaymentDate,
    SUM(Amount) AS DailyTotal
FROM Payments
WHERE PaymentDate >= DATEADD(DAY, -30, GETDATE())
GROUP BY CAST(PaymentDate AS DATE)
ORDER BY PaymentDate DESC;

✔ Stored Procedure – Add Payment
CREATE PROCEDURE usp_AddPayment
    @OrderNumber NVARCHAR(50),
    @Amount DECIMAL(18,2),
    @Method NVARCHAR(50),
    @Reference NVARCHAR(100)
AS
BEGIN
    DECLARE @OrderID INT;

    SELECT @OrderID = OrderID
    FROM SalesOrders
    WHERE OrderNumber = @OrderNumber;

    INSERT INTO Payments (OrderID, Amount, Method, Reference, Status)
    VALUES (@OrderID, @Amount, @Method, @Reference, 'Pending');

    SELECT 
        O.NetAmount,
        SUM(P.Amount) AS TotalPaid,
        O.NetAmount - SUM(P.Amount) AS Outstanding
    FROM SalesOrders O
    LEFT JOIN Payments P ON O.OrderID = P.OrderID
    WHERE O.OrderID = @OrderID
    GROUP BY O.NetAmount;
END;

** Key Insights Generated**

Identified high-risk outstanding orders

Measured real-time cash flow

Monitored retailer payment behavior

Flagged delivered-but-unpaid transactions

Built financial automation logic

** Business Impact**

This project simulates:

✔ Revenue reconciliation
✔ Credit risk monitoring
✔ Cash flow tracking
✔ Financial exposure control
✔ Payment lifecycle management
