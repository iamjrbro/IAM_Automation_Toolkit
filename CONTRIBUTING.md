# Contributing

1. Keep audit scripts read-only.
2. Add a short purpose, requested Graph scopes, example command, and known limitations for each script.
3. Do not include real tenant data, tokens, secrets, or personal data in commits or tests.
4. Add Pester coverage for reusable logic and report output.
5. Run `Invoke-Pester .\tests` and `Invoke-ScriptAnalyzer -Path .\src -Recurse` before submitting changes.
6. Prefer small, reviewable pull requests and document permission changes.
