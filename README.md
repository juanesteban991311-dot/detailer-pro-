# detailer-pro-
full-stacj app for professional detailing servivces in florida 
-- Clientes
create table customers (
  id uuid primary key default gen_random_uuid(),
  full_name text not null,
  phone text not null,
  email text,
  preferred_channel text check (preferred_channel in ('sms','whatsapp','email')) default 'sms',
  created_at timestamptz default now()
);

-- Vehículos
create table vehicles (
  id uuid primary key default gen_random_uuid(),
  customer_id uuid references customers(id) on delete cascade,
  year int,
  make text,
  model text,
  color text,
  plate text,
  notes text
);

-- Servicios (catálogo)
create table services (
  id uuid primary key default gen_random_uuid(),
  slug text unique not null,
  name text not null,
  description text,
  base_price_cents int not null,
  duration_min int not null,
  category text, -- exterior, interior, correction, coating, add-on
  is_active boolean default true
);

-- Citas / Jobs
create table jobs (
  id uuid primary key default gen_random_uuid(),
  customer_id uuid references customers(id),
  vehicle_id uuid references vehicles(id),
  scheduled_at timestamptz not null,
  status text check (status in ('pending_payment','booked','checked_in','in_progress','awaiting_pickup','completed','canceled')) default 'pending_payment',
  location text, -- shop o mobile service address
  notes text,
  total_cents int default 0,
  stripe_payment_intent text,
  created_at timestamptz default now()
);

-- Ítems del job (servicios seleccionados)
create table job_items (
  id uuid primary key default gen_random_uuid(),
  job_id uuid references jobs(id) on delete cascade,
  service_id uuid references services(id),
  qty int default 1,
  price_cents int not null
);

-- Eventos de estado (para disparar mensajes)
create table job_events (
  id uuid primary key default gen_random_uuid(),
  job_id uuid references jobs(id) on delete cascade,
  type text check (type in ('created','payment_succeeded','checked_in','stage_exterior','stage_interior','stage_correction','ready_for_pickup','completed','canceled')),
  meta jsonb,
  created_at timestamptz default now()
);
