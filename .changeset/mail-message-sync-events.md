---
'@nocobase/app-plugin-mail': minor
---

Add Server-only queue-backed subscriptions for committed synchronization inserts. The application provides `@nocobase/queue` through its existing QueueServiceProvider; Mail does not create its own queue service or persist event-log/sequence tables. Keep existing Mail synchronization, sending and frontend invalidation semantics unchanged.

Listeners receive a cancellation signal and asynchronous unsubscription must be awaited during shutdown. Queue workers await listeners and own their configured retries and retention; Redis instances compete for work rather than broadcasting to every instance. Keep business effects idempotent. Publication happens after commit without an event outbox, so failed acceptance or process termination in that window can lose events; neither listener nor publication failures retry committed synchronization. There are no incremental event-read or checkpoint APIs, and existing messages are not automatically backfilled.

Cross-dialect validation also hardens legacy localized-draft attachment lookup: external Provider part IDs are no longer compared with a native UUID upload column. Database test fixtures now use valid deterministic UUIDs on every dialect.

This release also edits the released `202609030001_create_mail_tables` migration because its outbound-attachment and push-subscription unique indexes have never succeeded on MySQL with utf8mb4 and prevent every later migration from running. Limit the generated storage-key column to 668 characters and the Provider push-subscription identifier to 413 characters so both existing composite unique indexes fit MySQL's 3072-byte limit without prefixing or weakening uniqueness. Existing installations that already applied this migration should expect a checksum warning on the next `nocobase db apply` and clear it with `nocobase db repair`; repair updates the recorded checksum and does not shrink their existing columns or remove attachment data.
