## What this changes

<!-- One or two sentences. Link the issue it fixes, e.g. "Closes #123". -->

## Checklist

- [ ] `dart format --output=none --set-exit-if-changed .` and `dart analyze --fatal-infos` are clean
- [ ] `dart test` passes, and `dart test --platform chrome` if `lib/` changed
- [ ] Closes an issue labelled `accepted`; at most 400 changed lines; no `// ignore:` (checked)
- [ ] Changes to `lib/` or `bin/` come with a test (checked)
- [ ] If this fixes a bug: there is a synthetic fixture in `test/support/` that fails without the fix
- [ ] Parsers still never throw on input and every read stays bounded
- [ ] User-visible change: `CHANGELOG.md` entry and README updated
- [ ] Linked issue:

### AI use

<!-- Using AI is welcome. Please tick one, naming the tool and model if you used AI; a reminder appears if it is missing. See CONTRIBUTING.md. -->

- [ ] None
- [ ] Some (tool or agent, and model: ...)
- [ ] Most (tool or agent, and model: ...)
- [ ] I have read and tested every line.
