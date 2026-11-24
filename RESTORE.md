# Як відновити базу Dream Team з копії

Копії лежать у `weekly/` (12 останніх неділь) і `monthly/` (перша копія кожного місяця).
Кожна — зашифрований архів `.tgz.gpg`: `public.dump` (усі таблиці застосунку) і `auth.dump` (логіни).

## 1. Розшифрувати (на комп'ютері з gpg і PostgreSQL 17 client)

```bash
gpg -d -o backup.tgz weekly/2026-11-29.tgz.gpg     # попросить пароль шифрування
tar -xzf backup.tgz                                 # → public.dump, auth.dump
```

## 2а. Відновити ОДНУ таблицю або рядки (найчастіший випадок)

Подивитись, що є в копії:
```bash
pg_restore --list public.dump | grep "TABLE DATA"
```
Витягти таблицю в SQL-файл і вже з нього взяти потрібні рядки:
```bash
pg_restore --data-only --table=guests -f guests.sql public.dump
```

## 2б. Повністю відновити в НОВИЙ проєкт Supabase

1. Створити новий проєкт, у SQL Editor увімкнути розширення, як у старому (pg_cron, pg_net, vault).
2. Логіни: `pg_restore --data-only -d "$NEW_DB_URL" auth.dump`
3. Таблиці: `pg_restore --no-owner --no-privileges -d "$NEW_DB_URL" public.dump`
4. Знову внести токени Nuga (`select set_salon_nuga_token(...)`) — вони в копію не потрапляють.
5. Запустити `scripts/push-schedule-setup.sql` і `scripts/nuga-sync-setup.sql` (розклад), оновити ключі у Vercel.

Не відновлюйте поверх робочої бази без потреби — спершу 2а.

## Перевірка, що копії робляться

GitHub → Actions → «DB backup»: зелена галочка щонеділі. Якщо копія не вдалась, GitHub надішле лист.
Раз на квартал варто розшифрувати одну копію (крок 1) — переконатися, що пароль правильний.
