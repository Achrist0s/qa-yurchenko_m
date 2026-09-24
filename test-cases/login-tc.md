\# Test cases: Login



\## TC-01 — Successful login with valid credentials



\*\*Preconditions:\*\* Registered user account exists.

\*\*Steps:\*\*

1\. Open the login page.

2\. Enter a valid username and password.

3\. Click "Login".



\*\*Expected result:\*\* User is redirected to the dashboard/catalog; login form disappears.



\## TC-02 — Login with non-existent username



\*\*Preconditions:\*\* None.

\*\*Steps:\*\*

1\. Open the login page.

2\. Enter a username that does not exist and any password.

3\. Click "Login".



\*\*Expected result:\*\* A specific error message is shown (e.g. "User not found"); no access is granted.

