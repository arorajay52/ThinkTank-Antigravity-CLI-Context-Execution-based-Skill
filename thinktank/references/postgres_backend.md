# Postgres & Supabase Backend Guidelines

This document details database architecture, Row-Level Security (RLS), custom triggers, RPC functions, and optimization patterns. Consult this file **only** when working on database schemas or backend-logic procedures.

---

## 1. Row-Level Security (RLS)

Always secure database tables at the engine level. No table should exist without RLS enabled.

### Enabling RLS:
```sql
ALTER TABLE public.table_name ENABLE ROW LEVEL SECURITY;
```

### RLS Policies Patterns:
*   **Public Read access**:
    ```sql
    CREATE POLICY "Allow public read access" ON public.table_name
      FOR SELECT USING (true);
    ```
*   **User-Bound Owner access (Read/Write)**:
    ```sql
    CREATE POLICY "Allow individual write access" ON public.table_name
      FOR ALL USING (auth.uid() = user_id) WITH CHECK (auth.uid() = user_id);
    ```
*   **Admin-only access**:
    ```sql
    CREATE POLICY "Allow admins all access" ON public.table_name
      FOR ALL TO authenticated USING (
        EXISTS (
          SELECT 1 FROM public.user_profiles
          WHERE id = auth.uid() AND role = 'admin'
        )
      );
    ```

---

## 2. Remote Procedure Call (RPC) Functions

Functions executed via client APIs (e.g., `supabase.rpc()`) must be built with strict security contexts.

### Security Definer vs. Invoker:
*   **`SECURITY INVOKER` (Default/Preferred)**: Runs with the privileges of the calling user. Safest context for standard user queries.
*   **`SECURITY DEFINER`**: Runs with the privileges of the function's creator (often admin/super-user). Use only when database triggers or actions require elevated permissions.

### CRITICAL Security Rule for DEFINER Functions:
Always explicitly set a safe `search_path` to prevent search-path hijacking vulnerabilities. Never omit this setting.
```sql
CREATE OR REPLACE FUNCTION public.custom_admin_operation(target_id UUID)
RETURNS VOID
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path = public, pg_temp -- Protects schemas from hijacking
AS $$
BEGIN
  -- Database logic here
END;
$$;
```

---

## 3. Triggers & Automation

Use database triggers to synchronize profiles, audit actions, or compute metrics automatically during write transactions.

### Creating a Trigger Function & Trigger:
```sql
-- 1. Create Trigger Function
CREATE OR REPLACE FUNCTION public.handle_new_user_sync()
RETURNS TRIGGER
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path = public, pg_temp
AS $$
BEGIN
  INSERT INTO public.user_profiles (id, email, name)
  VALUES (new.id, new.email, new.raw_user_meta_data->>'name');
  RETURN NEW;
END;
$$;

-- 2. Bind Trigger to Table
CREATE OR REPLACE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION public.handle_new_user_sync();
```

---

## 4. Performance & Indexing

*   **Foreign Key Indexes**: Add indexes to every foreign key column used in joins or filter operations.
    ```sql
    CREATE INDEX idx_table_name_user_id ON public.table_name(user_id);
    ```
*   **Composite Indexes**: For queries filtering by multiple fields concurrently (e.g. `WHERE user_id = X AND status = Y`), create a composite index:
    ```sql
    CREATE INDEX idx_table_user_status ON public.table_name(user_id, status);
    ```
*   **Cascade Deletes**: Ensure foreign keys that depend on user accounts or parent items clean up automatically using `ON DELETE CASCADE` to prevent orphaned rows.
