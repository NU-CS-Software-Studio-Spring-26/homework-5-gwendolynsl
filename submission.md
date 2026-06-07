# HW5 Submission

## Links

- Repo: [homework-5-gwendolynsl](https://github.com/NU-CS-Software-Studio-Spring-26/homework-5-gwendolynsl)
-.cursorignore: [.cursorignore](https://github.com/NU-CS-Software-Studio-Spring-26/homework-5-gwendolynsl/blob/main/.cursorignore)
- AGENTS.md: [AGENTS.md](https://github.com/NU-CS-Software-Studio-Spring-26/homework-5-gwendolynsl/blob/main/AGENTS.md)
- .cursor/rules/rails-conventions.mdc: [rails-conventions.mdc](https://github.com/NU-CS-Software-Studio-Spring-26/homework-5-gwendolynsl/blob/main/.cursor/rules/rails-conventions.mdc)
- .cursor/rules/security.mdc: [security.mdc](https://github.com/NU-CS-Software-Studio-Spring-26/homework-5-gwendolynsl/blob/main/.cursor/rules/security.mdc)



## Part 3 — Mode prompting

### Ask mode

Prompt: Where in this codebase is the create posts currently implemented. Specifically how it posts and renders errors? Cite the exact files and line numbers. Do not propose changes

Files + line numbers returned: config/routes.rb (line 2)
resources :todos → POST /todos maps to TodosController#create
app/controllers/todos_controller.rb
new (lines 14–16), create (lines 23–35), todo_params (lines 74–76)
app/models/todo.rb (line 2)
validates :description, presence: true — drives save failure and errors
app/views/todos/index.html.erb (line 16)
“New todo” link → new_todo_path
app/views/todos/new.html.erb (line 5)
Renders the form for a new record
app/views/todos/_form.html.erb
form_with(model: todo) POSTs to /todos; lines 2–11 render validation errors
app/views/todos/show.html.erb (line 1)
Success redirect target; shows flash notice
app/views/todos/_todo.html.erb
Partial rendered on show after successful create
app/views/todos/show.json.jbuilder (line 1)
JSON body on successful create (render :show)
app/views/todos/_todo.json.jbuilder (lines 1–2)
JSON partial used by show.json.jbuilder

Verified (real or hallucinated?): It was not hallucinated paths these are real paths some have realitively little to do with post creation but they are kinda relevant.



### Plan mode

Prompt:I want to change the create post behavior so that if a todo fails validation, the error messages are displayed in a dedicated error partial instead of inline in _form.html.erb. Propose a plan as a numbered list of changes, including files to edit, and new tests to add. Do not write code.

Plan returned: 
Numbered changes





Create [app/views/todos/_errors.html.erb](app/views/todos/_errors.html.erb) (new file)





Move the error markup currently in [_form.html.erb](app/views/todos/_form.html.erb) lines 2–11 into this partial.



Accept local todo; wrap in if todo.errors.any?.



Preserve existing content: red wrapper, pluralize(todo.errors.count, "error") heading, <ul> of error.full_message.



Add id="todo_errors" on the wrapper div so tests can target it reliably.



Edit [app/views/todos/_form.html.erb](app/views/todos/_form.html.erb)





Remove the inline error block (lines 2–11).



Leave only the form_with fields and submit button — form is input-only.



Edit [app/views/todos/new.html.erb](app/views/todos/new.html.erb)





Insert render "errors", todo: @todo above the existing render "form", todo: @todo (line 5).



Errors appear between the <h1> and the form on failed create.



Edit [app/views/todos/edit.html.erb](app/views/todos/edit.html.erb) (recommended)





Same change as step 3: render "errors" above "form".



Required because [update](app/controllers/todos_controller.rb) also re-renders :edit on failure (line 44) and shares _form.html.erb; without this, update errors would disappear.



No changes to controller, model, or routes





[TodosController#create](app/controllers/todos_controller.rb) / #update failure paths already correct.



Todo model validation unchanged.



JSON error responses (render json: @todo.errors) unaffected — HTML-only refactor.

New tests to add





Controller test — failed create in [test/controllers/todos_controller_test.rb](test/controllers/todos_controller_test.rb)





Name: "should not create todo with blank description".



POST todos_url, params: { todo: { description: "" } }.



Assert: :unprocessable_content (422), Todo.count unchanged, response body includes "Description can't be blank" and "prohibited this todo from being saved".



Controller test — failed update in same file (recommended)





Name: "should not update todo with blank description".



PATCH todo_url(@todo), params: { todo: { description: "" } }.



Assert: 422, validation message in body, @todo.reload.description unchanged.



System test — failed create in browser in [test/system/todos_test.rb](test/system/todos_test.rb)





Visit new todo page, leave Description blank, click "Create Todo".



Assert error text visible (e.g. "Description can't be blank"), success notice absent, still on new page ("New todo" heading present).



System test — failed update in browser in same file (optional, mirrors step 7)





Visit edit, clear Description, submit.



Assert error visible, update success message absent.

Verification





Run bin/rails test and bin/rails test:system — existing success-path tests must still pass.



Manually: blank submit on /todos/new shows errors above (not inside) the form; valid submit still redirects with green notice.
**My edits to the plan:**
- Combine steps 3 and 4 since they are the same changes
- Remove optional system test for failed update as unnecessary
- Be more specific with the verification step 


### Agent mode

**Prompt:**
Create a new partial at app/views/todos/_errors.html.erb. It should contain the error markup that's currently in _form.html.erb lines 2-11 — keep the same content but add id='todo_errors' to the outer div so tests can target it. The partial should accept todo as a local variable. Don't touch any other files yet.

**Commit link:**
https://github.com/NU-CS-Software-Studio-Spring-26/homework-5-gwendolynsl/commit/53f8e90709e96b570208294c449c83ae1073723b
---

### Bad → Good prompt rewrite

**Bad prompt:** fix the bug in todos

**Good prompt:**
Context: app/views/todos/_form.html.erb, app/controllers/todos_controller.rb, app/views/todos/_errors.html.erb
Task: Display validation errors in a dedicated _errors.html.erb partial instead of inline in the form
Expected: Errors appear above the form when description is blank
Actual: Errors are currently inline inside _form.html.erb lines 2-11
Constraints: Only touch view files, no new gems, follow existing ERB partial conventions
Done when: bin/rails test passes and submitting a blank todo on /todos/new shows errors above the form

---

## Part 4 — Turbo Streams

### My explanation of Turbo Streams
Turbo Streams is basically modifying one or more of the actions and only changing specific elements of the browser it can do things like append, prepend, replace, update, remove, before and after (which is just inserting an inline above or right below the last todo in the list in this case)

### What I verified against the Turbo Streams handbook
I cross referenced the actions with the handbook they all matched but it does look like cursor forgot to mention to me the "replace" command

## Workflow

### Submission
in my words:
As a user I would like to be able to mark a todo as high priority or atleast higher priority as other tasks so I can see the tasks I want to prioritize first. I would also like to be able to easily mark these priorities so I can see them visually and not have to do too much looking to see if I thought that task was supposed to be prioritized. 

Acceptance criteria
Todo gets a high_priority boolean attribute.
On the todos index, every row has a visible toggle (star icon, flag emoji, or text button)
showing the current priority state for that todo.
Clicking the toggle sends a request that flips the priority and returns a Turbo Stream that
updates only that row (or only the toggle button). The rest of the page must not re-render.
Verified in Chrome / Firefox DevTools, Network tab: the response Content-Type is
text/vnd.turbo-stream.html , and the request Accept header includes the same MIME
type.
At least one automated test covers the toggle.

### PR URL
#### Plan
Skip optional steps
(a) Migration and model attribute





Generate a migration in [db/migrate/](db/migrate/) (e.g. AddHighPriorityToTodos) following the pattern of [db/migrate/20260519180933_add_due_date_to_todo.rb](db/migrate/20260519180933_add_due_date_to_todo.rb).





Add high_priority as :boolean to the todos table.



Use default: false, null: false so existing rows and new records have a defined value.



Run bin/rails db:migrate (and db:test:prepare before tests).



Update [app/models/todo.rb](app/models/todo.rb) — no validation required for a boolean toggle; optionally add a small instance method (e.g. toggle_high_priority!) that flips and saves, keeping flip logic out of the controller. Not strictly required.



Update test fixtures in [test/fixtures/todos.yml](test/fixtures/todos.yml) — set explicit high_priority: false (or true on one fixture) so tests have predictable starting state.



Optional: add high_priority to [app/views/todos/_todo.json.jbuilder](app/views/todos/_todo.json.jbuilder) if JSON consumers should see the flag (not required for the Turbo Stream feature).



(b) Route and controller action





Add a member route in [config/routes.rb](config/routes.rb) under the existing resources :todos (line 2):





patch :toggle_priority as a member action → PATCH /todos/:id/toggle_priority → TodosController#toggle_priority.



Route helper: toggle_priority_todo_path(todo).



Add toggle_priority action in [app/controllers/todos_controller.rb](app/controllers/todos_controller.rb):





Add :toggle_priority to before_action :set_todo (currently line 2, only %i[ show edit update destroy ]).



Flip @todo.high_priority (toggle boolean, then save / update!, or call model helper from step 2).



Wrap in respond_to:





format.turbo_stream — primary path; implicit render of toggle_priority.turbo_stream.erb.



format.html { redirect_to todos_path } — fallback for non-Turbo clients (matches how other actions redirect on HTML).



Do not add high_priority to todo_params — the toggle is a dedicated action, not mass-assignment from the general update form (avoids opening update to priority changes via strong params).



Add controller test in [test/controllers/todos_controller_test.rb](test/controllers/todos_controller_test.rb):





PATCH toggle_priority_todo_url(@todo) with headers: { Accept: "text/vnd.turbo-stream.html" } (or equivalent).



Assert :success, @todo.reload.high_priority flipped, response body includes turbo-stream and dom_id(@todo) target.



Assert MIME type / content type is text/vnd.turbo-stream.html.



(c) Turbo Stream view and toggle button in partial





Create [app/views/todos/toggle_priority.turbo_stream.erb](app/views/todos/toggle_priority.turbo_stream.erb) — first .turbo_stream.erb in the project:





Use turbo_stream.replace dom_id(@todo) (or "#{dom_id(@todo)}") to replace the row wrapper.



Inside the block, render @todo (or render partial: "todos/todo", locals: { todo: @todo }) to re-render [_todo.html.erb](app/views/todos/_todo.html.erb) with the updated high_priority state.



Response MIME type will be text/vnd.turbo-stream.html automatically.



Add toggle button to [app/views/todos/_todo.html.erb](app/views/todos/_todo.html.erb):





Use button_to with method: :patch and toggle_priority_todo_path(todo) (Rails UJS/Turbo handles the PATCH).





Label or styling should reflect current state (e.g. "Mark high priority" vs "Mark normal" based on todo.high_priority?).



Optionally show priority visually (text, emoji, or CSS class) so the user sees the flip after the stream replaces the row.



No inline <script> — per project conventions, JS stays in Stimulus if needed (not required for basic button_to + Turbo Stream).



No changes required to [app/views/todos/index.html.erb](app/views/todos/index.html.erb) for the stream to work — it already renders <%= render todo %> inside #todos (line 9); each partial carries its own dom_id and button.



Add system test in [test/system/todos_test.rb](test/system/todos_test.rb) (optional but recommended):





Visit index, click toggle on a row, assert button label or priority indicator changes without full page navigation (e.g. still on index, h1 "Todos" present, priority text updated).



Verification





bin/rails db:migrate then bin/rails test and bin/rails test:system.



Manual: visit /todos, click toggle — row updates in place, no full reload; refresh confirms DB persisted.

## Tests
#### Plan
Skip optional steps
(a) Migration and model attribute





Generate a migration in [db/migrate/](db/migrate/) (e.g. AddHighPriorityToTodos) following the pattern of [db/migrate/20260519180933_add_due_date_to_todo.rb](db/migrate/20260519180933_add_due_date_to_todo.rb).





Add high_priority as :boolean to the todos table.



Use default: false, null: false so existing rows and new records have a defined value.



Run bin/rails db:migrate (and db:test:prepare before tests).



Update [app/models/todo.rb](app/models/todo.rb) — no validation required for a boolean toggle; optionally add a small instance method (e.g. toggle_high_priority!) that flips and saves, keeping flip logic out of the controller. Not strictly required.



Update test fixtures in [test/fixtures/todos.yml](test/fixtures/todos.yml) — set explicit high_priority: false (or true on one fixture) so tests have predictable starting state.



Optional: add high_priority to [app/views/todos/_todo.json.jbuilder](app/views/todos/_todo.json.jbuilder) if JSON consumers should see the flag (not required for the Turbo Stream feature).



(b) Route and controller action





Add a member route in [config/routes.rb](config/routes.rb) under the existing resources :todos (line 2):





patch :toggle_priority as a member action → PATCH /todos/:id/toggle_priority → TodosController#toggle_priority.



Route helper: toggle_priority_todo_path(todo).



Add toggle_priority action in [app/controllers/todos_controller.rb](app/controllers/todos_controller.rb):





Add :toggle_priority to before_action :set_todo (currently line 2, only %i[ show edit update destroy ]).



Flip @todo.high_priority (toggle boolean, then save / update!, or call model helper from step 2).



Wrap in respond_to:





format.turbo_stream — primary path; implicit render of toggle_priority.turbo_stream.erb.



format.html { redirect_to todos_path } — fallback for non-Turbo clients (matches how other actions redirect on HTML).



Do not add high_priority to todo_params — the toggle is a dedicated action, not mass-assignment from the general update form (avoids opening update to priority changes via strong params).



Add controller test in [test/controllers/todos_controller_test.rb](test/controllers/todos_controller_test.rb):





PATCH toggle_priority_todo_url(@todo) with headers: { Accept: "text/vnd.turbo-stream.html" } (or equivalent).



Assert :success, @todo.reload.high_priority flipped, response body includes turbo-stream and dom_id(@todo) target.



Assert MIME type / content type is text/vnd.turbo-stream.html.



(c) Turbo Stream view and toggle button in partial





Create [app/views/todos/toggle_priority.turbo_stream.erb](app/views/todos/toggle_priority.turbo_stream.erb) — first .turbo_stream.erb in the project:





Use turbo_stream.replace dom_id(@todo) (or "#{dom_id(@todo)}") to replace the row wrapper.



Inside the block, render @todo (or render partial: "todos/todo", locals: { todo: @todo }) to re-render [_todo.html.erb](app/views/todos/_todo.html.erb) with the updated high_priority state.



Response MIME type will be text/vnd.turbo-stream.html automatically.



Add toggle button to [app/views/todos/_todo.html.erb](app/views/todos/_todo.html.erb):





Use button_to with method: :patch and toggle_priority_todo_path(todo) (Rails UJS/Turbo handles the PATCH).





Label or styling should reflect current state (e.g. "Mark high priority" vs "Mark normal" based on todo.high_priority?).



Optionally show priority visually (text, emoji, or CSS class) so the user sees the flip after the stream replaces the row.



No inline <script> — per project conventions, JS stays in Stimulus if needed (not required for basic button_to + Turbo Stream).



No changes required to [app/views/todos/index.html.erb](app/views/todos/index.html.erb) for the stream to work — it already renders <%= render todo %> inside #todos (line 9); each partial carries its own dom_id and button.



Add system test in [test/system/todos_test.rb](test/system/todos_test.rb) (optional but recommended):





Visit index, click toggle on a row, assert button label or priority indicator changes without full page navigation (e.g. still on index, h1 "Todos" present, priority text updated).



Verification





bin/rails db:migrate then bin/rails test and bin/rails test:system.



Manual: visit /todos, click toggle — row updates in place, no full reload; refresh confirms DB persisted.

## Tests
All 7 tests passed (11 assertions, 0 failures).

Finished in 2.731481s
7 runs, 11 assertions, 0 failures, 0 errors, 0 skips
To get them running locally, two setup steps were needed first:

Pending migration — ran db:migrate for AddHighPriorityToTodos, then db:test:prepare.
Bundler platform — Gemfile.lock was updated with x64-mingw-ucrt so gems resolve on Windows (this changed Gemfile.lock).
If you only want the test command on a machine that's already set up:

ruby bin/rails test test/controllers/todos_controller_test.rb
