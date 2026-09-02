## What Is WatchGuard Cloud?

WatchGuard Cloud is a cloud-based platform used to monitor and manage WatchGuard products, devices, and WatchGuard services from one place.

The introductory video presents these areas:

- **Dashboard** — overview of activated products and services.
- **Monitor** — reports, logs, and other monitoring-related options.
- **Configure** — configuration options for products and services.
- **Shared configurations** — create one configuration and apply it to multiple devices or services.
- **Help, notifications, alarms, and administration** — additional management and support areas.

For a Subscriber account with activated devices or services, the Dashboard includes **Home** and **Threats** tabs. Widgets show aggregated information and link to the relevant detailed area.

> A Service Provider account has a different start page. The video refers learners to the separate *WatchGuard Cloud for Service Providers* course for that view.

## Partner vs End User

| Common term | WatchGuard Cloud account type | Role |
|---|---|---|
| Partner | **Tier-1 Service Provider** | Allocates devices, security services, and licenses to its own account or Subscriber accounts. |
| End User | **Tier-1 Subscriber** | Manages devices and services in its own account. |

Subscriber accounts typically represent a Partner’s customers.

### Account Delegation

An End User can delegate its account to a Partner. This lets the Partner manage the End User’s inventory.

Partners can also request managed access for an account that needs assistance. The End User must approve that request.

> **Memory line:** A Service Provider allocates and can manage customer inventory; a Subscriber manages its own inventory unless it delegates access.

## Logging In to WatchGuard Cloud

### Direct Login

All WatchGuard account types can use:

```text
https://cloud.watchguard.com
```

The user account must not be disabled.

### Login Through the WatchGuard Portal

Eligible Tier-1 Service Provider and Tier-1 Subscriber users can also access WatchGuard Cloud through:

```text
watchguard.com
→ My WatchGuard
→ Support Center
→ WatchGuard Cloud
```

The video’s takeaway states that this portal route is not available for accounts subscribed underneath a Tier-1 Service Provider. Those accounts should use `cloud.watchguard.com`.

### Authentication

The default authentication method is username and password. AuthPoint multi-factor authentication can also be prompted.

> **Source caution:** The supplied course transcript does not clearly state the exact condition that triggers MFA. Do not infer a detailed MFA policy from this video alone.

## Help and WatchGuard Support

### Context-Sensitive Help

In WatchGuard Cloud, select the **Help** icon (question mark) to access resources, including the Help Center.

The Help Center is context-sensitive: it opens documentation for the WatchGuard Cloud page currently being viewed.

### Contact Support

For accounts that can log in to `watchguard.com`:

```text
Support
→ Support
→ Contact Support
```

Available options:

- **Support by Phone** — country-specific phone numbers.
- **Support Online** — opens **My Cases** when you are signed in.

You can also access cases through:

```text
Support Center
→ My WatchGuard
→ Manage Cases
```

### Creating a Support Case

1. Describe the issue.
2. Select the applicable **Case Record Type**, **Product Family**, and other required information.
3. Select **Continue**.
4. Review the AI-generated answer and suggested resources.
5. If the resources do not resolve the issue, create a case for WatchGuard Support follow-up.

### Support Shortcut in WatchGuard Cloud

Some accounts show a headset/telephone icon in the top row of WatchGuard Cloud. Selecting it redirects to the Contact Support page.

This icon is only shown when the account has access to the WatchGuard portal.

## Key Takeaways

- WatchGuard Cloud centralizes monitoring and management of WatchGuard devices, products, and services.
- A Partner is a **Tier-1 Service Provider**.
- An End User is a **Tier-1 Subscriber**.
- Service Providers can allocate inventory and manage Subscriber inventory when access is delegated or approved.
- Use `cloud.watchguard.com` for direct login.
- The Dashboard provides an overview of activated products and services.
- The Help icon provides context-sensitive documentation.
- Support is available through phone contacts or online case management, subject to WatchGuard portal access.
