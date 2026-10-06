# BowlPOS

Cloud restaurant POS for Costa Rica. Servers take dine-in, takeaway, delivery, and phone tickets. The kitchen sees the same ticket. Three restaurants run it, about 200–300 orders a day, with peaks near 900.

**The source is not mine and stays private.** Repos live under the [Bowlpos](https://github.com/Bowlpos) org (`pos`, `admin`, `DB`, and related apps). Demo on request. This page does not contain their code.

The commits in the clones I read are not under my GitHub account. The architecture below is what their database schema actually implements, plus the operating numbers above from the restaurants.

## Architecture

Opposite of an offline register. The ticket lives in shared Postgres, so every station sees the same order. The floor needs internet. No custom hardware: a tablet or a browser.

```
staff POS (TanStack Start)          owner admin (TanStack Start)
  floor, takeaway, delivery,          menus, modifiers, customers,
  phone, checkout, kitchen            orders, end-of-night report
        \                            /
         \                          /
          Postgres (Supabase)
            create_order              SECURITY DEFINER RPC
            next_receipt_number       locks the receipt sequence
            sync_order_pos_lines      edits an open check
            transfer_dine_in_order    moves a check between tables
            get_restaurant_sales_report
          RLS scoped by restaurant
```

Staff do not insert into `orders` from the client. `create_order` is `SECURITY DEFINER`: it checks the caller, writes the ticket, and takes the next receipt number from `next_receipt_number`, which locks the sequence row so two terminals cannot mint the same number.

`sync_order_pos_lines` updates an open check and skips lines the kitchen already started. `transfer_dine_in_order` moves a dine-in check between tables. Sales totals go through `get_restaurant_sales_report` so the aggregation stays in Postgres.

Order statuses used by the product: `pending` → `confirmed` → `in_progress` → `ready` → `served` → `completed`.

An Electron shell can load the hosted POS. Order rows still live in Supabase, not on the PC.

## What I am not publishing

Their schema, their staff UI, and their admin app. Links above are to the org, not a mirror.

## License

This note is mine. Their product is not open source. All rights reserved.
