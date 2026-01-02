---
ADR: 0002
author: matmair
status: "proposed"
created: "2025-10-26"
decided: "yyyy-mm-dd"
date: "2025-10-26"
type: "architecture"
---

# Permissions and Access Control Revamp

## General Overview

To allow precise segmentation of all things (as requested multiple times, in #7718 for example) 2 new concepts are introduced:

- Workspaces
- Profiles

These can be seen as extensions of the existing concepts of Project Codes and Groups (in their usage for permission scoping).

## New Concepts

### Workspaces

Workspaces can be thought of similar to organizations in GitHub, Groups in GitLab or Organizational Units in Active Directory. They borrow some names/ideas from these concepts, but are tailored to the needs of InvenTree and are not a superset.

#### Workspace Business Rules

1. Workspaces support tree-like nesting, similar to part categories
2. Workspaces have attributes to make navigating and organisation for the user easier. Each must have a name and slug and can have a short name, description, icon and color, an owner (user or group) and a responsible (user or group). The later two might be used for targeting notifications
3. There is the default workspace "Default" (always pk 1, can not be deleted), which will be the default workspace for all models supporting workspaces
4. Each object supporting workspaces is assigned to exactly one workspace
5. Users can be a member of multiple workspaces
6. A user can only work in one workspace at a time / API call and can only create interactions between objects in that workspace or global objects (not assignable to any workspace)
7. Plugins can ship models that support workspaces or chose to stay global (available in all workspaces) - which is the default
8. API paths for all models/objects/actions that support workspaces will be optionally extended by a workspace identifier, so that all queries are scoped to the required workspace. If no workspace is provided for a workspace-scoped endpoint, the "Default" workspace is assumed.
9. Usage of the workspace feature can be enabled in a global system setting; disabling it will hide all UI features related to workspaces. Internally, workspaces will still be assigned to all new/manipulated objects with the "Default" workspace. Existing workspace assignments will be preserved, but ignored.
10. Disabling the workspace feature after it has been used would hide all objects not assigned to the "Default" workspace - therefore, I am considering adding a wizard to the Admin Center that would allow re-assigning all objects to the "Default" workspace before disabling the feature.

Workspaces would be implemented as a single workspace model and by adding a generic foreign key to all models that support workspaces.

#### Workspace Usage

Workspaces might be organized like this:

```text
- Default (default : 1)
- My Space Adventure (msa : 2)
  - Suits (msa/suit : 3)
    - First Eval Generation ( msa/suit/first : 4)
    - Andromda Generation (msa/suit/andromeda : 5)
    - Mars and Beyond Generation (msa/suit/mars : 6)
  - Rockets ( msa/rocket : 7)
  - Habitats ( msa/habitat : 8)
- Special Project X (spx : 9)
```

Being assigned access to "My Space Adventure" would give access to workspaces 2,3,4,5,6,7,8
Working in workspace 5 "msa/suit/andromeda" would only show objects to workspace 5
Working in workspace 2 "msa" would only show objects assigned to workspaces 2

### Profiles

Profiles are a way to represent a certain configuration area of InvenTree, there can be a number of different profiles types, but every object is only assigned to one profile per profile type at a time.

#### Profile Business Rules

1. Profile types are defined globally
2. Plugins can define new profile types
3. Profiles are instances of a single profile type
4. Profiles can have attributes, depending on their profile type
5. Each object can only be assigned to one profile per profile type
6. Default Profiles can be inherited (details to be defined) - so behavior can be defined at a higher level (e.g. part category) and inherited by all child objects unless overridden
7. Profiles can be "owned" by a user or group to allow controlling configuration of certain profile types

Model structure:

- ProfileType
  - name
  - description
  - source (core, plugin, unknown)
  - plugin (nullable FK to Plugin model)
- Profile
  - profile_type (FK to ProfileType)
  - name
  - description
  - settings (JSONField for profile specific settings)
  - owner (nullable FK to User or Group)
- ProfileAssignment
  - profile (FK to Profile)
  - content_type (Generic FK to any model supporting profiles)
  - object_id (Generic FK to any model supporting profiles)
  - inherited (bool, if the assignment is inherited from a parent object, e.g. part category)
  - inheritance_source (nullable Generic FK to the object the profile is inherited from)

#### Profile Usage

Profiles might be used for:

- Setting behavioral options for certain models (e.g. Part behavior profile, Supplier behavior profile)
- Access control (e.g. defining which actions are allowed on a model for users assigned to a certain profile)
- UI configuration (e.g. defining which fields are visible/editable for a model for users)
- Required parameters

In the first iteration, I plan to use ProfileAssigments as a lookup target for permission checks, making a gradual transition from classic Django permissions to Profile based permission scoping possible.

## Changes current things

### Existing Roles

The role system would be preserved as is for reverse compatibility, but definitions of roles would move to the individual models / apps where possible to remove the coupling between `users` / and the apps (`parts`, `stock`, etc) as much as possible.

### New Roles model?

I think it could make sense to have a new Roles model that defines roles as a model and that would make it usable by plugins as well.

### Expand permission information in the API schema

The API schema would be expanded to include information about which permissions are required for each endpoint. I did a small prototype of this in #10628.

### Extending docs / adding a best practices guide

The documentation should be extended to include information about how the permission system works and how to use it effectively. A best practices guide could be added to help users understand how to set up permissions in a way that is secure and is intended by the developers.

## Things I want to avoid / think might cause a problem

- Moving project codes into workspaces - project codes were introduces with another stated purpose and I want to avoid confusion
- Using project codes as a way to segment permissions - projects codes are very simple objects and would need far reaching changes to be used for permission scoping, bluring the line to workspaces
- Sharing stock items between workspaces - this would make the API more complex
- Using any dynamic scoping that is not pre-defined - I want to avoid having to calculate permissions on the fly as much as possible for performance and audit logging (#9996) reasons
- The ability for users to easily move objects between workspaces - this should be a deliberate action that requires admin access and is not done lightly (also because it is probably api-expensive)

## Usecases

### UC1: Marcus is a libarian
(1) He can create and modify components.
(2) But he cannot delete them.
(3) He also cannot add supplierparts and prices.

### UC2: Harold is working in the purchasing department
(1) He adds suppliers and their prices to the parts.
(2) But he cannot modify the parts or the parameters.
(3) He can also create and edit purchase orders.

### UC3: Monika is a designer
(1) She can view all parts, suppliers and prices because her job is also the create cost efficient designs.
(2) But she has no edit rights at all.

UC4: Barbara is a student worker supporting Monica.
(1) She can also see all the components.
(2) But not the prices because this might leak too much data.

UC5: Max is a tester working on the Astro project
(1) He can add and modify test results to all parts related to the Astro project.
(2) He can also see the results from the Moonshot project for comparison but he cannot edit them.
(3) When Max goes on vacation for two weeks he introduces Oliver as his deputy who usually works on Moonshot. For the time of Max vacation Oliver can edit Moonshot and Astro parts.

UC6: Bruce is working in the warehouse.
(1) He can shuffle around stock items and boxes between locations.
(2) But he cannot create new locations. This is up to Marvin who makes the floor plan of the warehouse.

UC7: Mike works in the factory.
(1) He can create parts that are manufactured, add serial numbers, and other data.
(2) But he cannot add or modify parts for design as Marcus can.

UC 8: Morgan is the administrator. He can do everything.

## Definitions

### Global

Something system wide, there is only one value / setting / instance for or of it. Can not be segmented any further. Example: System Settings, Users, Groups

#### User

A human actor interacting with the InvenTree system, represented by a User object, typically using the frontend

#### Actor

An entity that interacts with the system. This can be a user, automated system, workarea access station

### Model

A Django model

### Object

An instance of a Django model

### Action

An operation that can be performed via the API. Either on an object, a collection of objects, a model or against the global system
