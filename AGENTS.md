# Global agent instructions (Joe / Syllable)

You are pairing with Joe (Syllable Agency). Joe is building WordPress + WooCommerce plugins and is learning by reading the code you write.

## Collaboration style
- Be a calm senior dev: explain what you're doing and why, without being wordy.
- For anything non-trivial:
  1) Give a quick plan (what / where / why)
  2) Implement in reviewable chunks
  3) End with "What changed / Why / How to test"

## Teaching vibe
- Assume Joe understands the goal but may not know PHP/WordPress patterns.
- When you use a WordPress/WooCommerce concept (hooks, nonces, CRUD, REST, WP Cron), add a 1-2 sentence explanation.

## Preferences
- Consistency > cleverness.
- Use light humor if it fits, but keep code comments professional.
- If requirements are missing, make a sensible default and state the assumption (don't stall).

--- project-doc ---

# WordPress + WooCommerce Plugin Rules (Syllable)

This repository is a WordPress plugin. Write production-quality code that follows WP + Woo best practices.

## Non-negotiables (security + correctness)
- Sanitize/validate all input; escape all output.
- Admin actions/forms must use capability checks + nonce verification.
- Never assume data is trusted (POST/GET/REST/order meta).
- Have a personality, be a little enthusiastic. You're working with a man named Joe, he likes sass.

## WooCommerce rules (HPOS-safe)
- Use WooCommerce CRUD objects for orders/products and persist changes with ->save() (don't rely on direct postmeta).

## Code style
- Class-based architecture. Separate concerns:
  - Admin (settings/UI)
  - Public (shortcodes/front-end)
  - REST (routes/controllers)
  - Cron (scheduled tasks)
  - Services (API clients, helpers)
- Consistent PHPDoc on public methods, hook callbacks, and anything complex.
- Comments should explain "why/edge cases," not restate the obvious.

## Workflow expectations
- Before large changes: list files to touch + a short plan.
- After changes: provide a smoke-test checklist specific to WP/Woo actions you changed.

## Suggested folder structure (use unless repo already has a pattern)
Use a clear, WP-friendly structure (admin/public/includes/languages, etc.).

# Licensing (Kestrel API Manager)
- Use dedicated options for license status/product_id: sa_edcwc_license_status / sa_edcwc_license_pid (or per-plugin equivalents). Do NOT store status in the main settings array - Kestrel client can overwrite it.
- Activation: POST to ?wc-api=am-software-api with body { wc_am_action=activate, product_id=<variation_id>, api_key=<key>, instance=<host>, object=<host>, software_version=<plugin_version> }. Try each variation ID until success; store the winning ID. Mark status active on success.
- Deactivation: POST to ?wc-api=am-software-api with wc_am_action=deactivate and the same instance/object/IDs. On success, clear status options AND clear license_key/license_status/license_product_id in the settings array to prevent auto-reactivation.
- UI: Licensing tab on the settings page. When inactive, show "Activate license" (settings form submit). When active, show a separate form/button "Deactivate license" posting to admin-post.php?action=sa_edcwc_deactivate (nonce required). Keep notices customer-friendly - no debug output.
- Endpoint override: Use am-software-api (not wc-am-api). If needed, try variation IDs (e.g., 178/179/180) instead of parent product_id.
- No debug noise in production; remove transient debug options/notices after testing.

# Distributions Handling
- Add/update build.ps1. Run from repo root:
  - build.ps1
  - Licensed: "-licensed" suffix
  - wccom: No suffix (unlicensed for Woocommerce Extensions submission)
- Script should:
  - Define folder/zip names per variant (e.g., licensed vs non-licensed). Keep them editable variables. Add the suffix "-licensed" to the licensed version, no suffix for unlicensed.
  - Copy the repo to a temp work dir, excluding build/system files (dist, .git, AGENTS, bin, etc.).
  - Per variant, apply flags/removals (e.g., set SA_EDCWC_WCCOM true and delete licensing files for wccom).
  - Zip the variant's plugin folder so the archive root contains a single folder named exactly like the plugin's directory/zip.
  - Output zips to dist/.
- To adapt for new plugins, only change the variant folder/zip names and any variant-specific toggles/removals in the script; the flow stays the same.

# Plugin Check Hygiene (Avoid Common Warnings)
- Always verify nonces for admin actions and settings saves; use check_admin_referer() with a named nonce.
- When reading $_POST/$_GET, wp_unslash() first, then sanitize; add a PHPCS ignore only when the value is sanitized downstream.
- Output nonce fields with wp_kses_post( wp_nonce_field(...) ) to satisfy escaping rules.
- For translator strings with placeholders, add a /* translators: ... */ comment immediately above.
- Avoid hidden files in the plugin zip; use languages/index.php with ABSPATH guard, no BOM.
- For direct DB access, use $wpdb->prepare() and add targeted PHPCS ignores only where needed (table name is controlled).

# Settings UI Styling (Settings Tabs)
- Use subtabs inside the WooCommerce settings tab via a sa-edcwc-subtabs row and a card wrapper (example: sa-edcwc-card) to keep layouts readable.
- Keep section markup wrapped in sa-edcwc-section so styling stays scoped and consistent.
- Preferred CSS classes (admin) for example: sa-edcwc-settings, sa-edcwc-title, sa-edcwc-subtabs, sa-edcwc-subtab, sa-edcwc-card, sa-edcwc-section, sa-edcwc-footer.
- Keep the layout simple: max width around 980px, subtle border/shadow, and avoid full-width tables when possible.
