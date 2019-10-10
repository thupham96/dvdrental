### Query 1: What were the total number of rentals for each family movie category?

SELECT

    f.title film_title,

    c.name category_name,

    COUNT(r.rental_id) rental_count

FROM

    category c

    JOIN film_category fc

    ON c.category_id = fc.category_id

    JOIN film f

    ON fc.film_id = f.film_id

    JOIN inventory i

    ON f.film_id = i.film_id

    JOIN rental r

    ON i.inventory_id = r.inventory_id

WHERE c.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music')

GROUP BY 1, 2

ORDER BY 2, 1;


### Query 2: How do the two stores compare in their count of rentals?

SELECT

    store.store_id,

    DATE_PART('month', rental.rental_date) rental_month,

    DATE_PART('year', rental.rental_date) rental_year,

    COUNT(*) count_rentals

FROM store

    JOIN staff

    ON store.store_id = staff.store_id

    JOIN payment

    ON staff.staff_id = payment.staff_id

    JOIN rental

    ON payment.rental_id = rental.rental_id

GROUP BY 1, 2, 3

ORDER BY 3, 2;


### Query 3: Who are the top 10 paying customers and what are their total payment amount for each month?

WITH t1 AS (

    SELECT
        customer.customer_id,

        CONCAT(customer.first_name, ' ', customer.last_name) full_name,

        SUM(payment.amount) sum_amount

    FROM customer

    JOIN payment

        ON customer.customer_id = payment.customer_id

    GROUP BY 1, 2

    ORDER BY 3 DESC

    LIMIT 10)

SELECT
    t1.full_name,

    DATE_TRUNC('month', payment.payment_date) pay_mon,

    SUM(payment.amount) pay_amount,

    COUNT(*) pay_countpermon

FROM t1

    JOIN customer

    ON customer.customer_id = t1.customer_id

    JOIN payment

    ON payment.customer_id = customer.customer_id

GROUP BY 1, 2

ORDER BY 1, 2;


### Query 4: Who are the top 10 paying customers and how do their payment amount of each month compare to the previous one?

WITH t1 AS (

  SELECT

      customer.customer_id,

      CONCAT(customer.first_name, ' ', customer.last_name) full_name,

      SUM(payment.amount) sum_amount

  FROM customer

      JOIN payment

      ON customer.customer_id = payment.customer_id

  GROUP BY 1, 2

  ORDER BY 3 DESC

  LIMIT 10)

  SELECT

      t1.full_name full_name,

      DATE_TRUNC('month', payment.payment_date) pay_mon,

      SUM(payment.amount) pay_amount,

      LEAD(SUM(payment.amount)) OVER (PARTITION BY t1.full_name ORDER BY
      DATE_TRUNC('month', payment.payment_date)) lead,

      LEAD(SUM(payment.amount)) OVER (PARTITION BY t1.full_name ORDER BY

      DATE_TRUNC('month', payment.payment_date)) - SUM(payment.amount) difference

  FROM t1

      JOIN customer

      ON customer.customer_id = t1.customer_id

      JOIN payment

      ON payment.customer_id = customer.customer_id

  GROUP BY 1, 2

  ORDER BY 1, 2;
