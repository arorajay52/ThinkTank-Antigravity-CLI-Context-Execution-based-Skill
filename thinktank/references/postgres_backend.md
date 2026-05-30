# Postgres & Supabase Backend Guidelines (Verified Best Practices)

This document contains verified security and performance best practices for Postgres database engines and Supabase backend layers.

---

## 1. Security Definer Functions & Search Path Protection

By default, functions in PostgreSQL inherit the `search_path` of the calling user. In `SECURITY DEFINER` functions, this presents a critical vulnerability where malicious users can hijack execution by manipulating their search path to point to mock tables or procedures.

### The Standard Fix: Pin `search_path = ''` (Empty String)
*   **Vulnerability Prevention**: Always set `SET search_path = ''` when creating a `SECURITY DEFINER` function.
*   **Fully Qualified Names**: Forcing `search_path` to empty requires all database object references inside the function body to be fully qualified (e.g. `public.my_table`, `auth.users`).

```sql
-- ✅ SECURE: Pinning search_path to empty and using fully qualified names
CREATE OR REPLACE FUNCTION public.handle_user_registration()
RETURNS TRIGGER
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path = '' -- Critical security pin
AS $$
BEGIN
  INSERT INTO public.user_profiles (id, email, is_admin)
  VALUES (new.id, new.email, false);
  RETURN NEW;
END;
$$;
```

---

## 2. Invoker vs. Definer Context

*   **Prefer `SECURITY INVOKER` (Default)**: Always write functions as `SECURITY INVOKER` unless you explicitly need to bypass Row-Level Security (RLS) or execute procedures the caller has no direct rights to perform. Invoker functions naturally respect RLS policies during execution.
*   **Manual Checks in DEFINER**: If you must use `SECURITY DEFINER` (e.g., inside auth trigger events or admin actions), remember it **bypasses RLS entirely**. You must write manual security validation code inside the function body (e.g., verifying `auth.uid()` or role privileges).

---

## 3. Function Execute Permissions (Privilege Auditing)

By default, newly created database functions can be executed by the `PUBLIC` role (meaning any user or anonymous request can call them).
*   **Revoke Public Execute**: Always revoke public execution permissions on sensitive or administrative functions.
*   **Grant Explicitly**: Only grant `EXECUTE` privileges to specific authorized roles (e.g., `authenticated`, `service_role`).

```sql
-- Secure execution rights
REVOKE EXECUTE ON FUNCTION public.handle_user_registration() FROM PUBLIC;
REVOKE EXECUTE ON FUNCTION public.handle_user_registration() FROM anon;
GRANT EXECUTE ON FUNCTION public.handle_user_registration() TO service_role;
```

---

## 4. Row-Level Security (RLS) & Performance
*   **Advisors**: Use Dashboard Advisors (Database > Advisors) regularly to check for unsecured functions ("search path mutable") or tables missing RLS.
*   **Policy Optimization**: Avoid complex, slow queries in RLS `USING` clauses. RLS policies are evaluated for every single row returned by a query.
    *   If using functions in RLS policies, write them as helper views or wrap queries in a `SELECT` statement to allow Postgres to cache execution plans.
    *   Add B-Tree indexes to all columns referenced in RLS filter policies (e.g. `tenant_id`, `user_id`).
