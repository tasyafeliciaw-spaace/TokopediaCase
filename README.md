# TokopediaCaseSQL
The dataset that we used in this case contains 4 tables:

[customer_detail](https://drive.google.com/file/d/1vGkvIBs30dYItfbIplDc2eTJkLKrYaJS/view?usp=sharing)

[order_detail](https://drive.google.com/file/d/11Ekay2elqMoD6KpNxnKI9n3PJldRober/view?usp=sharing)

[payment_detail](https://drive.google.com/file/d/1Nl5qVB95tVq2qYizIFQV_nyYySw1WOWl/view?usp=sharing)

[sku_detail](https://drive.google.com/file/d/11AS-dlvpWdZBzLrw98rDFx0tk3PD1Hjf/view?usp=sharing)

In SQL, we use the dataset to:

**Analyze Transaction**

1. Identify category with highest sales in 2022


    a. Showing transaction value per category in 2022
    
        ```
           SELECT SUM(od.after_discount)nilai_transaksi, sd.category
           FROM order_detail od JOIN sku_detail sd ON od.idSku = sd.idSku
           WHERE od.is_valid = 1 AND order_date BETWEEN '2022-01-01' AND '2022-12-31'
           GROUP BY sd.category
           ORDER BY SUM(after_discount) DESC`
    
    
     b. Shows the total sum of all transactions in 2022
                
           SELECT SUM(after_discount)total_nilai_transaksi22 
           FROM order_detail`

2. Show transaction value growth (2021-2022) based on product category

   a. Create a temporary table with CTE to display data that can be used to compare the development of transaction values ​​per category from 2021-2022

        ```
        WITH totalTransaksi AS(
         SELECT SUM(after_discount) nilai_transaksi
         FROM order_detail
        )
        , transaksi2021 AS(
         SELECT sd.category, SUM(od.after_discount) nilai_transaksi
         FROM order_detail od LEFT JOIN sku_detail sd ON od.idSku = sd.idSku
         WHERE od.is_valid = 1 AND order_date BETWEEN '2021-01-01' AND '2021-12-31'
         GROUP BY sd.category
        )
        
        , transaksi2022 AS(
         SELECT sd.category, SUM(od.after_discount) nilai_transaksi
         FROM order_detail od LEFT JOIN sku_detail sd ON od.idSku = sd.idSku
         WHERE od.is_valid = 1 AND order_date BETWEEN '2022-01-01' AND '2022-12-31'
         GROUP BY sd.category
        )
        
        SELECT 
        COALESCE(t21.category,t22.category) category, 
        COALESCE (t21.nilai_transaksi, 0) transaksi_2021,
        COALESCE (t22.nilai_transaksi, 0) transaksi_2022,
        CASE --> membuat logic seperti if-else di Java
         WHEN COALESCE (t21.nilai_transaksi, 0) > COALESCE (t22.nilai_transaksi, 0) THEN 'Penurunan'
         WHEN COALESCE (t21.nilai_transaksi, 0) < COALESCE (t22.nilai_transaksi, 0) THEN 'Peningkatan'
         ELSE 'Stagnan'
         END AS status_grafik
        FROM transaksi2021 t21 FULL JOIN transaksi2022 t22 ON t21.category = t22.category
        ORDER BY  status_grafik ASC`

   b. View the largest transaction value per category each year
   
        SELECT 
        COALESCE(t21.category,t22.category) category, 
        COALESCE (t21.nilai_transaksi, 0) transaksi_2021,
        COALESCE (t22.nilai_transaksi, 0) transaksi_2022,
        COALESCE(t22.nilai_transaksi, 0) - COALESCE(t21.nilai_transaksi, 0) AS selisih_transaksi,
        CASE --> membuat logic seperti if-else di Java
         WHEN COALESCE (t21.nilai_transaksi, 0) >  COALESCE (t22.nilai_transaksi, 0) THEN 'Penurunan'
         WHEN COALESCE (t21.nilai_transaksi, 0) <  COALESCE (t22.nilai_transaksi, 0) THEN 'Peningkatan'
         ELSE 'Stagnan'
         END AS status_grafik
        FROM transaksi2021 t21 FULL JOIN transaksi2022 t22 ON t21.category = t22.category
        ORDER BY COALESCE (t21.nilai_transaksi,0) DESC


* using the COALESCE method so that NULL data can be handled by returning another value
  
3. Show top 5 payment method in 2022
   ```
   WITH top_5 AS(
     SELECT COUNT (DISTINCT od.idOrder) total_payment,
   CAST (pd.payment_method as varchar (500)) payment_method
   FROM order_detail od JOIN payment_detail pd
     ON od.idPayment = pd.idPayment WHERE od.is_valid = 1
     AND YEAR(od.order_date)='2022'
     GROUP BY CAST (pd.payment_method as varchar (500)) )

   SELECT TOP 5 total_payment, payment_method
   FROM top_5
   ORDER BY top_5.total_payment DESC

4. Sorting brand name based on the transaction value in Mobiles & Tablets category
   ```
   SELECT 
    CASE
        WHEN sd.sku_name LIKE '%Samsung%' THEN 'Samsung'
        WHEN sd.sku_name LIKE '%Apple%' OR LOWER(CAST(sd.sku_name AS VARCHAR)) LIKE '%iphone%'
            OR LOWER(CAST(sd.sku_name AS VARCHAR)) LIKE '%macbook%' THEN 'Apple'
        WHEN sd.sku_name LIKE '%Sony%' THEN 'Sony'
        WHEN sd.sku_name LIKE '%Huawei%' THEN 'Huawei'
        WHEN sd.sku_name LIKE '%Lenovo%' THEN 'Lenovo'
    END AS nama_produk, 
    SUM(od.after_discount) AS total_nilai_transaksi
   FROM order_detail od
   JOIN sku_detail sd ON od.idSku = sd.idSku

   WHERE od.is_valid = 1
     AND (
     sd.sku_name LIKE '%Samsung%' OR
     sd.sku_name LIKE '%Apple%' OR
     LOWER(CAST(sd.sku_name AS VARCHAR)) LIKE '%iphone%' OR
     LOWER(CAST(sd.sku_name AS VARCHAR)) LIKE '%macbook%' OR
     sd.sku_name LIKE '%Sony%' OR
     sd.sku_name LIKE '%Huawei%' OR
     sd.sku_name LIKE '%Lenovo%'
     )
    
     GROUP BY 
        CASE
            WHEN sd.sku_name LIKE '%Samsung%' THEN 'Samsung'
            WHEN sd.sku_name LIKE '%Apple%' OR LOWER(CAST(sd.sku_name AS VARCHAR)) LIKE '%iphone%'
                OR LOWER(CAST(sd.sku_name AS VARCHAR)) LIKE '%macbook%' THEN 'Apple'
            WHEN sd.sku_name LIKE '%Sony%' THEN 'Sony'
            WHEN sd.sku_name LIKE '%Huawei%' THEN 'Huawei'
            WHEN sd.sku_name LIKE '%Lenovo%' THEN 'Lenovo'
        END
     ORDER BY total_nilai_transaksi DESC


