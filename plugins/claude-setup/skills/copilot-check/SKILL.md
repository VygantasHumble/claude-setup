---
name: copilot-check
description: Use when asked to check Copilot comments, handle Copilot reviews, or resolve automated PR feedback on a pull request.
---

# Copilot Check

Review and triage GitHub Copilot review comments on the current branch's PR, then resolve them.

## When to Use

- User asks to check/handle Copilot comments on a PR
- User wants to resolve automated review feedback
- After pushing changes to a PR that has Copilot reviews enabled

## Workflow

1. **Find the PR number:**

   ```bash
   gh pr list --head $(git branch --show-current) --json number --jq '.[0].number'
   ```

2. **Fetch all review comments from Copilot:**

   ```bash
   gh api repos/{owner}/{repo}/pulls/{number}/comments \
     --jq '.[] | select(.user.login == "Copilot") | {id, path, line, body}'
   ```

   `{owner}` and `{repo}` are auto-resolved by `gh api`.

3. **Fetch unresolved thread IDs** (needed for resolving later):

   ```bash
   gh api graphql -f query='{
     repository(owner: "{OWNER}", name: "{REPO}") {
       pullRequest(number: {NUMBER}) {
         reviewThreads(first: 50) {
           nodes {
             id
             isResolved
             comments(first: 1) {
               nodes { body databaseId }
             }
           }
         }
       }
     }
   }' --jq '.data.repository.pullRequest.reviewThreads.nodes[]
     | select(.isResolved == false)
     | {threadId: .id, commentId: .comments.nodes[0].databaseId, body: (.comments.nodes[0].body[:80])}'
   ```

   Match each Copilot comment from step 2 to its thread using `commentId` / `databaseId`.

4. **Categorize each comment:**
   - **Already fixed** - the issue was addressed in a later commit
   - **Not relevant** - over-engineering, hypothetical concern, or doesn't apply to actual usage
   - **Relevant** - a real issue worth discussing or fixing

5. **Present a summary table to the user:**

   | # | File | Issue | Category |
   |---|------|-------|----------|

   Ask the user which "Relevant" items to fix vs dismiss.

6. **For items the user wants fixed:** make the code changes.

7. **Resolve all threads on GitHub:**
   - Reply only if adding useful context (1-2 sentences max). Skip replies for obvious dismissals.
   - Resolve each thread via GraphQL:

     ```bash
     gh api graphql -f query='mutation {
       resolveReviewThread(input: {threadId: "THREAD_NODE_ID"}) {
         thread { isResolved }
       }
     }'
     ```

## Common Mistakes

- **Wrong thread ID format** - The `threadId` for `resolveReviewThread` is the GraphQL node ID from step 3 (starts with `PRRT_`), not the REST API comment `id`.
- **Missing comments** - `pulls/{number}/comments` only returns review comments. Regular issue-style PR comments use a different endpoint (`issues/{number}/comments`). Copilot uses review comments.
- **Owner/repo in GraphQL** - Unlike REST (`{owner}/{repo}` auto-resolved), GraphQL queries need literal owner and repo names. Get them with `gh repo view --json owner,name`.