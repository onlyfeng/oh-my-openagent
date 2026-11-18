# src/hooks/ -- 56 Lifecycle Hooks Across 61 Dirs

**Generated:** 2026-11-18 (update pass)

## OVERVIEW

56 base registered hooks on default config (63 with team-mode; `monitor-status-injector` adds 1 more with `monitor.enabled` → 64 max), composed from 55 `index.ts` hook dirs (53 wired; `task-reminder/` and `ralph-loop/` unwired) plus 5 standalone hook `.ts` files at the top level (bash-file-read-guard, empty-task-response-detector, preemptive-compaction, session-notification, tool-output-truncator). 61 directories total = 55 with `index.ts` + 6 without (`shared/`, `team-session-events/`, 4 `zauc-mocks-*`/`zauc-sync-mocks`). All hooks follow `createXXXHook(deps) -> HookFunction` factories; 5-tier composition wired in `src/plugin/hooks/`. NEW since the last pass: `nativeEditionNudge`, `momusReviewPersistence`, `btwSideContextInjector`.

**Unwired WIP (do not wire casually):** `task-reminder/` (has `index.ts` + `createTaskReminderHook` but NOT exported from the barrel, NOT imported by any composer) and `ralph-loop/` (barrel-exported but imported by no composer; retained for migration, superseded by `goal/`).

## TIER COMPOSITION

| Tier | Composer | Base | With team-mode | Where |
|------|----------|------|----------------|-------|
| **Session** | `create-session-hooks.ts` | 24 | 24 | OpenCode session lifecycle + chat.params + chat.message |
| **Tool Guard** | `create-tool-guard-hooks.ts` | 18 | 19 | Pre/post tool execution (+1: `team-tool-gating`; NEW: `momusReviewPersistence`) |
| **Transform** | `create-transform-hooks.ts` | 5 | 7 | `experimental.chat.messages.transform` (+2 team-mode injectors; NEW base: `btwSideContextInjector`; `monitor-status-injector` is a further +1 gated on `monitor.enabled`, not team-mode) |
| **Continuation** | `create-continuation-hooks.ts` | 7 | 7 | Boulder/atlas/compaction/notification |
| **Skill** | `create-skill-hooks.ts` | 2 | 2 | Skill awareness (categorySkillReminder, autoSlashCommand) |
| **Direct event handlers** | `src/plugin/event.ts` | 0 | +4 | `team-session-events/` sub-files: `team-idle-wake-hint`, `team-lead-orphan-handler`, `team-member-error-handler`, `team-member-status-handler` |

Total exposed hooks: **56 base, 63 with team-mode, 64 with team-mode + monitor** (counts the 4 team-session-events handlers individually).

Hook name allowlist for `disabled_hooks`: the 58 `HookName` literals in [`src/config/schema/hooks.ts`](../config/schema/hooks.ts) `HookNameSchema` (now incl. `momus-review-persistence` and `native-edition-nudge`). NOT in the schema: `btwSideContextInjector` + `contextInjectorMessagesTransform` (built unconditionally) and the team-session-event sub-hooks, which activate together with `team_mode.enabled`.

## HOOK ROSTERS (name -> event)

- **Session (24):** preemptiveCompaction · sessionNotification (session.idle) · thinkMode · modelFallback (chat.params) · anthropicContextWindowLimitRecovery (session.error) · autoUpdateChecker · astGrepSgProvision · nativeEditionNudge (session.created) · agentUsageReminder · nonInteractiveEnv · ulwExecute · sisyphusJuniorNotepad · taskResumeInfo · noSisyphusGpt · noHephaestusNonGpt · hephaestusAgentsMdInjector · legacyPluginToast (chat.message) · interactiveBashSession (tool.execute) · editErrorRecovery · delegateTaskRetry (tool.execute.after) · prometheusMdOnly · questionLabelTruncator (tool.execute.before) · goal · runtimeFallback (event)
- **Tool Guard (18 base):** commentChecker · toolOutputTruncator · emptyTaskResponseDetector · hashlineReadEnhancer · jsonErrorRecovery · readImageResizer · fsyncSkipWarning (tool.execute.after) · directoryAgentsInjector · directoryReadmeInjector · rulesInjector · tasksTodowriteDisabler · writeExistingFileGuard · bashFileReadGuard · todoDescriptionOverride · webfetchRedirectGuard · notepadWriteGuard · planFormatValidator (tool.execute.before) · momusReviewPersistence (tool.execute.before + after)
- **Transform (5):** claudeCodeHooks · keywordDetector · contextInjectorMessagesTransform · btwSideContextInjector · toolPairValidator (messages.transform)
- **Continuation (7):** stopContinuationGuard (chat.message) · compactionContextInjector · compactionTodoPreserver (session.compacted) · todoContinuationEnforcer · unstableAgentBabysitter (session.idle) · backgroundNotificationHook · atlasHook (event)
- **Skill (2):** categorySkillReminder · autoSlashCommand (chat.message)

## CONVENTIONS

- Standard construction is `isHookEnabled(<name>) ? safeHook(<name>, factory) : null` -- fail-safe registration (`safeCreateHook` turns a throwing factory into `null`, never a broken plugin).
- Per-feature dir layout: `index.ts` barrel + implementation + colocated `*.test.ts` (bun:test, given/when/then style).
- `directoryAgentsInjector` self-disables when the running OpenCode is >= `OPENCODE_NATIVE_AGENTS_INJECTION_VERSION` (native support wins).

## ADDING A NEW HOOK

1. `mkdir src/hooks/{name}` + `index.ts` exporting `createXXXHook(deps)`; export it from the `src/hooks/index.ts` barrel
2. Pick the right tier:
   - Session lifecycle? → `create-session-hooks.ts`
   - Pre/post tool? → `create-tool-guard-hooks.ts`
   - Message transform? → `create-transform-hooks.ts`
   - Continuation/idle? → `create-continuation-hooks.ts`
   - Skill awareness? → `create-skill-hooks.ts`
   - Team-mode-only? → register inside the team-mode conditional block
3. Add hook name to [`config/schema/hooks.ts`](../config/schema/hooks.ts) `HookNameSchema` (unconditional hooks skip this)
4. Cover with co-located `*.test.ts` (given/when/then style)

## NOTES

- **Tier order matters within a phase:** within Session tier the registration order in `create-session-hooks.ts` determines invocation order -- earlier hooks see un-mutated input, later hooks see accumulated output.
- **Mock files** (`zauc-mocks-*`, `zauc-sync-mocks`) are NOT hooks. They are placed inside `src/hooks/` purely so `bun:test` discovers them with the hook test fixtures.
- **`atlasHook` vs `todoContinuationEnforcer`:** atlas handles boulder/ralph/subagent sessions, todoContinuationEnforcer handles the main Sisyphus session. Both fire on `session.idle` but check session type first.
- **`runtime-fallback` vs `model-fallback`:** runtime-fallback is reactive (after error); model-fallback is proactive (chat.params). They operate independently.
- **`goal` replaces `ralphLoop` (PR #6184):** `ralphLoop` was removed from `HookNameSchema` and `create-session-hooks.ts`; the `/ralph-loop`, `/ulw-loop`, `/cancel-ralph` builtin commands were removed. The `ralph-loop/` dir + `createRalphLoopHook` factory remain for migration (barrel export kept); `ralph_loop` config is a deprecated passthrough. See [`goal/AGENTS.md`](goal/AGENTS.md).
