create table public.recurring_transactions (
  id serial not null,
  user_id integer not null,
  category_id integer not null,
  type character varying(20) not null,
  amount numeric(10, 2) not null,
  description text null,
  frequency character varying(20) not null,
  start_date date not null,
  end_date date null,
  next_run_date date not null,
  last_run_date date null,
  is_active boolean not null default true,
  created_at timestamp without time zone null default CURRENT_TIMESTAMP,
  payment_source character varying(20) not null default 'online'::character varying,
  constraint recurring_transactions_pkey primary key (id),
  constraint recurring_transactions_category_id_fkey foreign KEY (category_id) references categories (id) on delete RESTRICT,
  constraint recurring_transactions_user_id_fkey foreign KEY (user_id) references users (id) on delete CASCADE,
  constraint recurring_transactions_amount_check check ((amount > (0)::numeric)),
  constraint recurring_transactions_payment_source_check check (
    (
      (payment_source)::text = any (
        (
          array[
            'cash'::character varying,
            'online'::character varying,
            'credit_card'::character varying
          ]
        )::text[]
      )
    )
  ),
  constraint recurring_transactions_type_check check (
    (
      (type)::text = any (
        (
          array[
            'income'::character varying,
            'expense'::character varying
          ]
        )::text[]
      )
    )
  ),
  constraint recurring_transactions_next_run_check check ((next_run_date >= start_date)),
  constraint recurring_transactions_date_check check (
    (
      (end_date is null)
      or (end_date >= start_date)
    )
  ),
  constraint recurring_transactions_frequency_check check (
    (
      (frequency)::text = any (
        (
          array[
            'daily'::character varying,
            'weekly'::character varying,
            'monthly'::character varying,
            'yearly'::character varying
          ]
        )::text[]
      )
    )
  )
) TABLESPACE pg_default;

create index IF not exists idx_recurring_transactions_user_id on public.recurring_transactions using btree (user_id) TABLESPACE pg_default;

create index IF not exists idx_recurring_transactions_category_id on public.recurring_transactions using btree (category_id) TABLESPACE pg_default;

create index IF not exists idx_recurring_transactions_next_run on public.recurring_transactions using btree (next_run_date) TABLESPACE pg_default
where
  (is_active = true);

create index IF not exists idx_recurring_transactions_active on public.recurring_transactions using btree (is_active) TABLESPACE pg_default;

create unique INDEX IF not exists unique_recurring_rule_per_user on public.recurring_transactions using btree (user_id, category_id, frequency, start_date) TABLESPACE pg_default;

create index IF not exists idx_recurring_transactions_payment_source on public.recurring_transactions using btree (payment_source) TABLESPACE pg_default;
