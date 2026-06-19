/**
 * Returns true if the current logged-in user is in the AD admin group
 * (ARADMIN), which grants edit rights on the "Global" keyword list.
 */
export function isGlobalKeywordAdmin() {
  try {
    const user = JSON.parse(sessionStorage.getItem('redactor_user') || '{}');
    if (typeof user.is_admin === 'boolean') return user.is_admin;
    const memberOf = user.member_of || [];
    return memberOf.some(g => String(g).toLowerCase().includes('aradmin'));
  } catch {
    return false;
  }
}
