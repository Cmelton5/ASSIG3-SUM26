Defect 1 - SQL Injection Authentication Bypass

Vulnerability and Why It Was Exploitable:
The login feature was vulnerable because user input was directly concatenated into the SQL query. This allowed an attacker to inject SQL commands into the username field and modify the query logic.
The vulnerable query was:
SELECT id, username FROM users
WHERE username = '<username>' AND password = '<password>'
Because the input was not treated as data, an attacker could close the username string and comment out the password check.

Payload Used and What It Did:
The payload used was:
curator' --
The payload closed the username string and used the SQL comment operator (--) to ignore the password condition. This caused the application to authenticate the curator account without requiring the correct password.

Fix Applied and Why It Works:
The query was changed to use a parameterized query:
const sql =
  `SELECT id, username FROM users ` +
  `WHERE username = ? AND password = ?`;

user = get(sql, [username, password]);
Parameterized queries treat user input as data instead of executable SQL. This prevents special characters from changing the structure of the query and stops SQL injection attacks.

------------------------------------------
Defect 2 - SQL Injection Data Extraction

Vulnerability and Why It Was Exploitable:
The search feature was vulnerable because the search term and sorting value were inserted directly into the SQL query. This allowed attackers to inject SQL commands and retrieve information from other database tables.
Payload Used and What It Did:
The payload used was:
%' UNION SELECT id, username, password, "x" FROM users --
This payload closed the original search string, added a UNION query that retrieved usernames and passwords from the users table, and commented out the remaining SQL query.
The result was that sensitive information from the users table was returned in the search results.

Fix Applied and Why It Works:
The search query was changed to use parameterized values:
const sql =
  `SELECT id, title, species, location FROM listings ` +
  `WHERE title LIKE ? OR species LIKE ? ` +
  `ORDER BY ${orderBy}`;
rows = all(sql, [pattern, pattern]);

The sorting value was also restricted using an allow-list:
const allowedSorts = {
  title: "title",
  species: "species",
  location: "location"
};

The search input is now treated as data, and the allowed sorting values prevent attackers from injecting SQL through the ORDER BY clause.

------------------------------------------
Defect 3 - Reflected XSS
Vulnerability and Why It Was Exploitable:
The search page was vulnerable because the user's search term was inserted directly into the HTML response without encoding.
The vulnerable section displayed:
Showing results for "${q}"
Because the value was not escaped, the browser interpreted the input as HTML instead of normal text.

Payload Used and What It Did:
The payload used was:
<alert>CRM was here</alert>
Before the fix, this input was reflected directly into the page. This demonstrated that attacker-controlled HTML could be inserted into the response.
Fix Applied and Why It Works:

The application now HTML-encodes user input before displaying it:
q
  .replaceAll('&', '&amp;')
  .replaceAll('<', '&lt;')
  .replaceAll('>', '&gt;')
  .replaceAll('"', '&quot;')
  .replaceAll("'", '&#39;')

Encoding special HTML characters prevents the browser from interpreting user input as HTML markup.

------------------------------------------
Defect 4 - Stored XSS in Comments

Vulnerability and Why It Was Exploitable:
The comments feature was vulnerable because stored comments were displayed directly in the HTML page without encoding.
The vulnerable code was:
<p class="comment-body">${c.body}</p>
Since comments were stored and displayed as HTML, attackers could save malicious content that executed whenever another user viewed the listing.

Payload Used and What It Did:
The payload used was:
<img src=x onerror='alert("xss_MARKER")'>

The invalid image caused the onerror event to execute. Because the comment was stored, the payload remained available and could execute for every visitor viewing the page.
Fix Applied and Why It Works:
The application now HTML-encodes comment content before displaying it.
Encoding characters such as <, >, and quotes prevents the browser from treating the comment as HTML or executable code.

------------------------------------------
Defect 5 - Missing Content Security Policy

Vulnerability and Why It Was Exploitable:
The application did not include a Content Security Policy (CSP) header. Without CSP, browsers had no additional restrictions preventing inline scripts or event handlers from executing if another vulnerability allowed malicious HTML injection.
The exploit test logged in using valid credentials:
username: curator
password: GreenThumb!Root#2024
The response was checked for security headers. Before the fix, the CSP header was missing.
Payload Used and What It Did:
No direct injection payload was used for this defect.
The exploit verified that security protections were missing by checking whether the application returned a CSP header and secure cookie attributes.
Fix Applied and Why It Works:
A middleware function was added to send a CSP header:
app.use((req, res, next) => {
  res.setHeader(
    'Content-Security-Policy',
    "default-src 'self'; script-src 'self'; style-src 'self'"
  );
  next();
});

This restricts scripts and styles to trusted application resources and blocks inline script execution.
