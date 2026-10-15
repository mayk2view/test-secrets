# Test scenarios (run after the ruleset is active)

Generate a fake token at test time (do NOT commit it to main). It has the github-pat shape gitleaks flags:
`echo "token=ghp_$(head -c 200 /dev/urandom | tr -dc 'A-Za-z0-9' | head -c 36)" > config.txt`

| # | Action | Expected |
|---|--------|----------|
| 1 | Branch `t1-leak`, add `config.txt` with `token=<fake token>`, open PR to main | check fails, merge blocked |
| 2 | Branch `t2-clean`, change README only, PR to main | check passes, merge allowed |
| 3 | Branch `t3-allow`, same fake token with a trailing `# gitleaks:allow` comment | passes |
| 4 | Branch `t4-testdir`, fake token inside `tests/data.txt` | passes (path allowlisted) |
| 5 | Branch `develop` created, PR `t1-leak` -> `develop` | check runs and fails, but merge NOT blocked (ruleset targets default branch only) |
| 6 | Push a commit directly to `main` | rejected by ruleset (no passing check on new commit) |
| 7 | Push `t1-leak` with no PR | nothing runs |

Note: public repos have GitHub push protection on by default; if it rejects the fake token push, use the bypass URL it prints ("used in tests").
