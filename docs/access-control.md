# Access control

Access control resources let tenants define reusable role bundles and organise
members into workspaces. Both routers inherit the tenant-aware base classes, so
authentication and authorization checks are applied automatically before CRUD
operations.【F:app/utils/usso_routes.py†L1-L24】

## Roles

### Responsibilities

* Model reusable permission bundles (scopes) that can be assigned to users,
  workspaces, or agent credentials.【F:app/apps/access_control/role/routes.py†L1-L39】
* Provide activation flags so roles can be phased out without deleting historical
  records.【F:app/apps/access_control/role/routes.py†L26-L39】
* Expose descriptive metadata (name, slug, description) for downstream UX
  surfaces.【F:app/apps/access_control/role/schemas.py†L1-L52】

### Endpoint map

| Method & path | Summary |
| --- | --- |
| `POST /roles` | Creates a new role with scoped permissions and optional workspace restrictions.【F:app/apps/access_control/role/routes.py†L1-L26】 |
| `GET /roles` | Lists roles with pagination so administrators can audit available scopes.【F:app/apps/access_control/role/routes.py†L1-L26】 |
| `GET /roles/{uid}` | Retrieves a single role, including assigned scopes and metadata.【F:app/apps/access_control/role/routes.py†L1-L26】 |
| `PATCH /roles/{uid}` | Updates scope membership or toggles `is_active` to retire a role.【F:app/apps/access_control/role/routes.py†L26-L39】 |
| `DELETE /roles/{uid}` | Removes a role when it is no longer required (inherits base delete).【F:app/apps/access_control/role/routes.py†L1-L26】 |

### Integration checklist

1. Use slugs to reference roles from configuration files or automation scripts;
   they remain stable even if display names change.【F:app/apps/access_control/role/schemas.py†L1-L52】
2. Keep scope bundles granular—combine them via roles rather than assigning large
   raw scope lists to users.
3. When deprecating access, toggle `is_active` first to block new assignments and
   then clean up existing users with the `/users` endpoints.【F:app/apps/identity/user/routes.py†L153-L178】

## Workspaces

### Responsibilities

* Segment tenant data into named workspaces with their own metadata payload and
  configuration settings.【F:app/apps/access_control/workspace/routes.py†L1-L56】
* Provide invitation flows that tie a workspace to role assignments and expiry
  windows.【F:app/apps/access_control/workspace/routes.py†L29-L56】【F:app/apps/access_control/workspace/schemas.py†L13-L63】
* Support pagination and search so large tenants can manage many workspaces
  efficiently.【F:app/apps/access_control/workspace/routes.py†L1-L29】

### Endpoint map

| Method & path | Summary |
| --- | --- |
| `POST /workspaces` | Creates a workspace with slug, metadata, and optional default roles.【F:app/apps/access_control/workspace/routes.py†L1-L29】 |
| `GET /workspaces` | Lists workspaces for assignment in other systems.【F:app/apps/access_control/workspace/routes.py†L1-L29】 |
| `GET /workspaces/{uid}` | Returns workspace metadata and configuration payloads.【F:app/apps/access_control/workspace/routes.py†L1-L29】 |
| `PATCH /workspaces/{uid}` | Updates metadata, branding, or default roles.【F:app/apps/access_control/workspace/routes.py†L29-L56】 |
| `DELETE /workspaces/{uid}` | Removes a workspace (inherited delete handler).【F:app/apps/access_control/workspace/routes.py†L1-L29】 |
| `POST /workspaces/{uid}/invitations` | Issues workspace invitations specifying target roles and expiry windows.【F:app/apps/access_control/workspace/routes.py†L29-L56】 |

### Integration checklist

1. Mirror workspace slugs in downstream products to link USSO access with your
   business domains.【F:app/apps/access_control/workspace/schemas.py†L1-L63】
2. Use invitations when onboarding external collaborators; the shared invitation
   schema captures email, roles, and expiry in one payload.【F:app/apps/access_control/workspace/schemas.py†L13-L63】
3. Combine workspace assignments with role bundles to achieve least privilege—
   issue narrow scopes per workspace rather than global admin roles.
=======
# Access Control Resources

Access control resources organize tenants into workspaces and assign reusable
role definitions. Both routers inherit the tenant-aware base so permissions are
enforced consistently.

## Roles (`/roles`)

Roles define reusable bundles of scopes and optional workspace affinity. Use them
to express high-level permissions (e.g., `admin`, `support`). 【F:app/apps/access_control/role/routes.py†L1-L11】【F:app/apps/access_control/role/schemas.py†L1-L10】

Common operations:

* `POST /api/sso/v1/roles` – Create a role with descriptive metadata and scope
  list.
* `GET /api/sso/v1/roles` – List roles to audit which scopes exist in a tenant.
* `PATCH /api/sso/v1/roles/{uid}` – Update scopes or deactivate a role to phase
  it out without deleting historical assignments.

Why this resource exists: central role definitions keep scope assignments
manageable as tenants scale their teams and integrate multiple applications.

## Workspaces (`/workspaces`)

Workspaces segment tenant data into sub-organizations and carry their own
metadata and settings payload. Invitations can be issued per workspace, allowing
role-based onboarding. 【F:app/apps/access_control/workspace/routes.py†L1-L10】【F:app/apps/access_control/workspace/schemas.py†L1-L21】

Key flows:

* `POST /api/sso/v1/workspaces` – Create a workspace with a human-friendly slug
  and optional branding.
* `GET /api/sso/v1/workspaces` – Enumerate available workspaces for assignment in
  other tools.
* `POST /api/sso/v1/workspaces/{uid}/invitations` – Issue workspace invitations
  with roles and expiry windows using the shared invitation schema. 【F:app/apps/access_control/workspace/schemas.py†L13-L21】

Why we have it: workspaces enable multi-team tenants to isolate access without
running independent SSO deployments.
