def _require_global_keyword_admin(group_id: str, user: dict) -> None:
    """Raise 403 if the target group is 'global' and the user is not an admin."""
    if (group_id or "").strip().lower() != "global":
        return
    member_of = user.get("member_of") or []
    is_admin = any("aradmin" in str(g).lower() for g in member_of)
    if not is_admin:
        raise HTTPException(
            status_code=403,
            detail={"error": "Only admins can modify the Global keyword list"},
        )
