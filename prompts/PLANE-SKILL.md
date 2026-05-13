---
name: plane
description: Use when creating, updating, listing, or managing Plane project management tasks, issues, states, labels, cycles, modules, or pages/docs. Auto-invoke on bug found, feature completed, task state change needed, or documentation required.
---

# Plane API Skill

## Setup
```bash
PROJECT_ID=$(cat .plane)
BASE="$PLANE_URL/api/v1/workspaces/$PLANE_WORKSPACE/projects/$PROJECT_ID"
HEADERS='-H "X-Api-Key: $PLANE_API_KEY" -H "Content-Type: application/json"'
```

## Workflow
Always fetch states first on a new session to get UUIDs:
```bash
curl -s "$BASE/states/" -H "X-Api-Key: $PLANE_API_KEY" | python3 -m json.tool
```

---

## Issues

### Create
```bash
curl -s -X POST "$BASE/issues/" \
  -H "X-Api-Key: $PLANE_API_KEY" -H "Content-Type: application/json" \
  -d '{"name":"title","description_html":"<p>details</p>","priority":"medium","state":"<state-uuid>"}'
```
Priority: `urgent` `high` `medium` `low` `none`

### List
```bash
curl -s "$BASE/issues/" -H "X-Api-Key: $PLANE_API_KEY"
```

### Update (state, priority, assignees)
```bash
curl -s -X PATCH "$BASE/issues/<uuid>/" \
  -H "X-Api-Key: $PLANE_API_KEY" -H "Content-Type: application/json" \
  -d '{"state":"<state-uuid>","priority":"high"}'
```

### Delete
```bash
curl -s -X DELETE "$BASE/issues/<uuid>/" -H "X-Api-Key: $PLANE_API_KEY"
```

### Add comment
```bash
curl -s -X POST "$BASE/issues/<uuid>/comments/" \
  -H "X-Api-Key: $PLANE_API_KEY" -H "Content-Type: application/json" \
  -d '{"comment_html":"<p>comment</p>"}'
```

### Add link
```bash
curl -s -X POST "$BASE/issues/<uuid>/links/" \
  -H "X-Api-Key: $PLANE_API_KEY" -H "Content-Type: application/json" \
  -d '{"title":"title","url":"https://..."}'
```

---

## Labels

### List
```bash
curl -s "$BASE/labels/" -H "X-Api-Key: $PLANE_API_KEY"
```

### Create
```bash
curl -s -X POST "$BASE/labels/" \
  -H "X-Api-Key: $PLANE_API_KEY" -H "Content-Type: application/json" \
  -d '{"name":"bug","color":"#ff0000"}'
```

---

## Cycles (Sprints)

> **Check first:** Cycles require `cycle_view: true` on the project. Verify with:
> `curl -s "$PLANE_URL/api/v1/workspaces/$PLANE_WORKSPACE/projects/$PROJECT_ID/" -H "X-Api-Key: $PLANE_API_KEY" | python3 -c "import sys,json; d=json.load(sys.stdin); print('cycle_view:', d['cycle_view'])"`
> Skip cycle operations silently if `false`.

### List
```bash
curl -s "$BASE/cycles/" -H "X-Api-Key: $PLANE_API_KEY"
```

### Create
```bash
curl -s -X POST "$BASE/cycles/" \
  -H "X-Api-Key: $PLANE_API_KEY" -H "Content-Type: application/json" \
  -d '{"name":"Sprint 1","start_date":"2025-01-01","end_date":"2025-01-14"}'
```

### Add issue to cycle
```bash
curl -s -X POST "$BASE/cycles/<uuid>/cycle-issues/" \
  -H "X-Api-Key: $PLANE_API_KEY" -H "Content-Type: application/json" \
  -d '{"issues":["<issue-uuid>"]}'
```

---

## Modules

> **Check first:** Modules require `module_view: true` on the project. Verify with:
> `curl -s "$PLANE_URL/api/v1/workspaces/$PLANE_WORKSPACE/projects/$PROJECT_ID/" -H "X-Api-Key: $PLANE_API_KEY" | python3 -c "import sys,json; d=json.load(sys.stdin); print('module_view:', d['module_view'])"`
> Skip module operations silently if `false`.

### List
```bash
curl -s "$BASE/modules/" -H "X-Api-Key: $PLANE_API_KEY"
```

### Create
```bash
curl -s -X POST "$BASE/modules/" \
  -H "X-Api-Key: $PLANE_API_KEY" -H "Content-Type: application/json" \
  -d '{"name":"Module Name","description":"description"}'
```

### Add issue to module
```bash
curl -s -X POST "$BASE/modules/<uuid>/module-issues/" \
  -H "X-Api-Key: $PLANE_API_KEY" -H "Content-Type: application/json" \
  -d '{"issues":["<issue-uuid>"]}'
```

---

## Members
```bash
curl -s "$BASE/members/" -H "X-Api-Key: $PLANE_API_KEY"
```

---

## Pages (Docs)

> **API limitations:** The pages list endpoint (`GET $BASE/pages/`) and page update (`PATCH/PUT`) are not supported on self-hosted Plane. Only Create and Read (individual page) are available.

### Create
```bash
curl -s -X POST "$BASE/pages/" \
  -H "X-Api-Key: $PLANE_API_KEY" -H "Content-Type: application/json" \
  -d '{"name":"Doc title","description_html":"<p>content</p>"}'
```

### Read (single page)
```bash
curl -s "$BASE/pages/<uuid>/" -H "X-Api-Key: $PLANE_API_KEY"
```

### Link a page to an issue
```bash
curl -s -X POST "$BASE/issues/<issue-uuid>/links/" \
  -H "X-Api-Key: $PLANE_API_KEY" -H "Content-Type: application/json" \
  -d '{"title":"Related doc","url":"https://plane.yourdomain.com/.../pages/<page-uuid>/"}'
```

---

## Gotchas
- Use UUIDs not sequence IDs (PROJ-1) in API calls
- Pipe output through `python3 -m json.tool` for readability
- Issue links endpoint is `/links/` not `/issue-links/` (verified on self-hosted)
- Pages: only Create and single-page Read work on self-hosted Plane; List/Update/Delete return 405
- Cycles and Modules require `cycle_view`/`module_view` to be enabled on the project — check before using
- Module `status` field is not a free-text string; omit it or leave empty when creating modules
