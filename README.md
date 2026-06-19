token = auth.issue_jwt(user_info)
is_admin = any("aradmin" in str(g).lower() for g in user_info.get("member_of", []))
user_info_with_sub = {**user_info, "sub": user_info.get("username", ""), "is_admin": is_admin}
return {
    "token":       token,
    "user":        user_info_with_sub,
    "auth_config": auth.get_auth_config(),
}
