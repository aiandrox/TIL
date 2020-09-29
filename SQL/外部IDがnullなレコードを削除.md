中間テーブル（coupon_plans）から、クーポンが存在しないレコードを削除したい。

SELECT

```sql
SELECT
  *
FROM
  coupon_plans
LEFT JOIN
  coupons
  ON (
    coupon_plans.coupon_id = coupons.id
  )
  WHERE
    coupons.id IS NULL
```

DELETE　エラーが出るやつ

```sql
DELETE
FROM
  coupon_plans
WHERE
  coupon_plans.id
  IN (
  SELECT
    coupon_plans.id 
  FROM
    coupon_plans
    LEFT JOIN
      coupons
    ON(
      coupon_plans.coupon_id = coupons.id)
    WHERE
      coupons.id IS NULL)
```

[You can't specify target table '\*\*\*' for update in FROM clause〜MySQLにて、サブクエリのみに適用されるエラーがある〜 \- 君は心理学者なのか？](https://karoten512.hatenablog.com/entry/2018/03/08/111917)

仮の名前（tmp）を付けてあげる

```sql
DELETE
FROM
  coupon_plans
WHERE
  coupon_plans.id
  IN (
  SELECT
    tmp.id 
  FROM(
    SELECT coupon_plans.id FROM
    coupon_plans
    LEFT JOIN
      coupons
    ON(
      coupon_plans.coupon_id = coupons.id)
    WHERE
      coupons.id IS NULL) as tmp)

```
