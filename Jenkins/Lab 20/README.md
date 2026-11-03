# Lab 20 – Jenkins Role-Based Authorization

## Objective

The goal of this lab was to configure role-based authorization in Jenkins and give different users different levels of access.

## What I Did

1. Used my existing Jenkins installation running on Docker Desktop.
2. Installed and enabled the Role-Based Authorization Strategy plugin.
3. Created two roles:
   - **admin:** Full access to Jenkins.
   - **readonly:** Can view Jenkins jobs but cannot modify or execute them.
4. Created two Jenkins users, `user-1` and `user-2`.
5. Assigned the admin role to `user-1` and the readonly role to `user-2`.
6. Tested access by logging in with both accounts.

## Role Configuration

| User   | Role     | Permissions                       |
| ------ | -------- | --------------------------------- |
| user-1 | admin    | Full Jenkins access               |
| user-2 | readonly | Overall/Read, Job/Read, View/Read |

I also kept my original administrator account to avoid losing access to Jenkins.

## Verification

**Admin User (user-1)**

Verified that the admin user could access Jenkins administration settings and manage jobs.

**Read-Only User (user-2)**

Verified that the read-only user could log in and view the Jenkins dashboard and existing jobs without administrative access.

The read-only role was configured without job build, configuration, or deletion permissions.
