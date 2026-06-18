def _get_user_info(search_conn: "ldap3.Connection", username: str) -> dict:
    """Fetch displayName, mail, memberOf, department from AD."""
    info: dict = {"username": username, "name": username, "mail": "",
                  "department": "", "member_of": []}
    if not _ldap_base_dn:
        return info
    try:
        safe_user = ldap3.utils.conv.escape_filter_chars(username)
        search_conn.search(
            search_base   = _ldap_base_dn,
            search_filter = f"(sAMAccountName={safe_user})",
            search_scope  = SUBTREE,
            attributes    = ["displayName", "mail", "memberOf", "department",
                              "sAMAccountName"],
        )
        if not search_conn.entries:
            log.warning("[auth] no AD entry found for %s", username)
            return info
        entry = search_conn.entries[0]
    except Exception as e:
        log.warning("[auth] user search failed: %s", e)
        return info

    # Each field is read independently — one missing/bad attribute must not
    # block the others, especially member_of which gates group access.
    try:
        raw_name = str(entry.displayName) if entry.displayName else ""
    except Exception:
        raw_name = ""
    raw_name = raw_name.strip()

    try:
        mail = str(entry.mail) if entry.mail else ""
    except Exception:
        mail = ""
    info["mail"] = mail

    try:
        dept = str(entry.department) if entry.department else ""
    except Exception:
        dept = ""
    info["department"] = dept

    try:
        info["member_of"] = [str(g) for g in (entry.memberOf or [])]
    except Exception as e:
        log.warning("[auth] memberOf read failed for %s: %s", username, e)
        info["member_of"] = []

    # Resolve display name: displayName -> derived from email local-part -> username
    if _has_real_name(raw_name):
        info["name"] = raw_name
    else:
        derived = _name_from_email(mail)
        info["name"] = derived or username

    return info


def _has_real_name(name: str) -> bool:
    """True if name contains at least one letter or digit (rejects '', ',', '-', etc.)."""
    return any(ch.isalnum() for ch in name)


def _name_from_email(mail: str) -> str:
    """Derive a display name from the email local-part before the first '.'.
    e.g. 'svp01.ams@istsbgendbca.com' -> 'Svp01'
    """
    if not mail or "@" not in mail:
        return ""
    local_part = mail.split("@", 1)[0]
    first_segment = local_part.split(".", 1)[0]
    if not first_segment:
        return ""
    return first_segment[:1].upper() + first_segment[1:]
