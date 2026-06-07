# AGENTS.md

## Stack

Rails 8.1 sample todo app (`TodoApp`), Ruby 3.4.1, SQLite (`storage/*.sqlite3` per `config/database.yml`).
View layer: ERB templates, Jbuilder for JSON, Hotwire (`turbo-rails`, `stimulus-rails`), importmap-rails, Propshaft CSS — no Bootstrap gem.
Tests: Rails default Minitest (`test/`), Capybara + Selenium for system tests — not RSpec.
Background jobs: Active Job (`ApplicationJob` in `app/jobs/`); `solid_queue`, `solid_cache`, and `solid_cable` gems — production adapters in `config/environments/production.rb` and `config/cable.yml`.

## Commands

Setup: `bin/setup` (bundle install, `db:prepare`, clear logs/tmp; execs `bin/dev` unless `--skip-server`).
Run dev server: `bin/dev` (wraps `bin/rails server`).
Test: `bin/rails test` (unit/controller); `bin/rails test:system` for system tests. CI: `bin/rails db:test:prepare test test:system`.
Lint / security: `bin/rubocop` (rubocop-rails-omakase); `bin/brakeman`.
Other: `bin/rails db:seed`, `bin/rails console`, `bin/rails routes`.

## Conventions

Naming: Rails plural REST resources (`TodosController`, `Todo` model, `app/views/todos/`).
Authorization: none yet — only `ApplicationController` with `allow_browser`; do not assume auth gems or filters exist.
Mutating actions use `respond_to` for HTML and JSON; read actions rely on ERB/Jbuilder templates. Use `params.expect(...)` for strong params (see `todo_params`, `set_todo` in `TodosController`).
Shared partials live in `app/views/<controller>/` with leading underscore (e.g. `_todo.html.erb`, `_form.html.erb`); collection render via `render todo`.
JavaScript belongs in Stimulus controllers under `app/javascript/controllers/`, pinned via `config/importmap.rb` — not inline in ERB.

## Don'ts

- Do not add gems without explicit approval (especially auth, CSS frameworks, or test runners).
- Do not put inline `<script>` tags or hand-written JavaScript inside ERB templates.
- Do not use `skip_before_action :verify_authenticity_token` or disable CSRF protection.
- Do not read or bypass files listed in `.cursorignore` (`.env`, `config/master.key`, credentials, `.kamal/secrets`, logs, `storage/`, etc.).
- Do not introduce RSpec — extend existing Minitest files under `test/`.
- Do not commit real secrets or production data; sample data belongs in `db/seeds.rb` only.
