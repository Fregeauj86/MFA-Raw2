// MFA
const http = require('http');
const url = require('url');
const querystring = require('querystring');

const users = {}; // username -> { password, mfaCode }

const server = http.createServer((req, res) => {
  const parsedUrl = url.parse(req.url);
  const pathname = parsedUrl.pathname;

  if (req.method === 'GET') {
    if (pathname === '/register') {
      res.end(`
        <h2>Register</h2>
        <form method="POST" action="/register">
          Username: <input name="username" /><br/>
          Password: <input type="password" name="password" /><br/>
          <button type="submit">Register</button>
        </form>
      `);
    } else if (pathname === '/login') {
      res.end(`
        <h2>Login</h2>
        <form method="POST" action="/login">
          Username: <input name="username" /><br/>
          Password: <input type="password" name="password" /><br/>
          <button type="submit">Login</button>
        </form>
      `);
    } else if (pathname === '/mfa') {
      res.end(`
        <h2>MFA Verification</h2>
        <form method="POST" action="/mfa">
          Enter code (Hint: 123456): <input name="code" /><br/>
          <button type="submit">Verify</button>
        </form>
      `);
    } else {
      res.end('404 Not Found');
    }
  }

  // Handle form submissions
  else if (req.method === 'POST') {
    let body = '';
    req.on('data', chunk => body += chunk);
    req.on('end', () => {
      const data = querystring.parse(body);

      if (pathname === '/register') {
        const { username, password } = data;
        if (users[username]) return res.end('User already exists');
        users[username] = { password, mfaCode: '123456' }; // fake fixed MFA code
        res.end('Registered successfully. <a href="/login">Login</a>');

      } else if (pathname === '/login') {
        const { username, password } = data;
        const user = users[username];
        if (!user || user.password !== password) {
          return res.end('Invalid username or password');
        }
        // Save username temporarily in "session" (in-memory)
        req.user = username;
        res.writeHead(302, { Location: '/mfa' });
        res.end();

      } else if (pathname === '/mfa') {
        const { code } = data;
        if (code === '123456') {
          res.end('✅ MFA Verified! You are logged in.');
        } else {
          res.end('❌ Wrong code. <a href="/mfa">Try again</a>');
        }

      } else {
        res.end('Unknown POST request');
      }
    });
  }
});

server.listen(3000, () => {
  console.log('Server running at http://localhost:3000');
});
