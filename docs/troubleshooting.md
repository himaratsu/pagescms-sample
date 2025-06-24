---
layout: default
title: Troubleshooting
nav_order: 5
description: "Common issues and solutions"
---

# Troubleshooting
{: .no_toc }

Find solutions to common problems and issues you might encounter.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Installation Issues

### Node.js Version Problems

**Problem**: Getting errors during installation related to Node.js version.

**Solution**:
1. Check your Node.js version:
   ```bash
   node --version
   ```
2. Ensure you're using Node.js 14 or higher
3. Use nvm to manage Node.js versions:
   ```bash
   # Install nvm (if not already installed)
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
   
   # Install and use Node.js 18
   nvm install 18
   nvm use 18
   ```

### Package Installation Failures

**Problem**: `npm install` or `yarn install` fails with permission errors.

**Solution**:
1. Try clearing npm/yarn cache:
   ```bash
   npm cache clean --force
   # or
   yarn cache clean
   ```

2. Delete `node_modules` and package-lock files:
   ```bash
   rm -rf node_modules package-lock.json
   npm install
   ```

3. For permission issues on macOS/Linux:
   ```bash
   sudo chown -R $(whoami) ~/.npm
   ```

## Database Connection Issues

### PostgreSQL Connection Failed

**Problem**: Cannot connect to PostgreSQL database.

**Symptoms**:
```
Error: connect ECONNREFUSED 127.0.0.1:5432
```

**Solutions**:

1. **Check if PostgreSQL is running**:
   ```bash
   # Ubuntu/Debian
   sudo systemctl status postgresql
   
   # macOS
   brew services list | grep postgresql
   ```

2. **Start PostgreSQL service**:
   ```bash
   # Ubuntu/Debian
   sudo systemctl start postgresql
   
   # macOS
   brew services start postgresql
   ```

3. **Verify connection settings**:
   ```bash
   psql -h localhost -p 5432 -U your_username -d your_database
   ```

4. **Check firewall settings** (if using remote database):
   ```bash
   telnet your_db_host 5432
   ```

### Database Migration Errors

**Problem**: Database migrations fail during startup.

**Solution**:
1. Check migration files for syntax errors
2. Ensure database user has proper permissions:
   ```sql
   GRANT ALL PRIVILEGES ON DATABASE your_db TO your_user;
   GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO your_user;
   ```
3. Run migrations manually:
   ```bash
   npm run migrate
   # or
   yarn migrate
   ```

## API Issues

### Authentication Errors

**Problem**: Getting 401 Unauthorized responses.

**Common Causes and Solutions**:

1. **Invalid API Key**:
   - Verify API key in environment variables
   - Check for extra spaces or characters
   - Ensure key hasn't expired

2. **Incorrect Header Format**:
   ```javascript
   // Wrong
   headers: { 'Authorization': 'your_api_key' }
   
   // Correct
   headers: { 'Authorization': 'Bearer your_api_key' }
   ```

3. **Missing Content-Type**:
   ```javascript
   headers: {
     'Authorization': 'Bearer your_api_key',
     'Content-Type': 'application/json'  // Don't forget this!
   }
   ```

### Rate Limit Exceeded

**Problem**: Getting 429 Too Many Requests errors.

**Solutions**:
1. Implement exponential backoff:
   ```javascript
   async function retryWithBackoff(fn, maxRetries = 3) {
     for (let i = 0; i < maxRetries; i++) {
       try {
         return await fn();
       } catch (error) {
         if (error.status === 429 && i < maxRetries - 1) {
           const delay = Math.pow(2, i) * 1000; // 1s, 2s, 4s
           await new Promise(resolve => setTimeout(resolve, delay));
           continue;
         }
         throw error;
       }
     }
   }
   ```

2. Check rate limit headers:
   ```javascript
   console.log('Rate limit remaining:', response.headers['x-ratelimit-remaining']);
   console.log('Rate limit reset:', response.headers['x-ratelimit-reset']);
   ```

## Performance Issues

### Slow API Responses

**Problem**: API requests taking too long to complete.

**Diagnostic Steps**:

1. **Check network latency**:
   ```bash
   ping api.example.com
   curl -w "@curl-format.txt" -o /dev/null -s "https://api.example.com/endpoint"
   ```

2. **Monitor database queries**:
   ```bash
   # Enable slow query log in PostgreSQL
   # Add to postgresql.conf:
   log_min_duration_statement = 1000  # Log queries taking > 1 second
   ```

3. **Profile your application**:
   ```javascript
   console.time('api-call');
   const result = await apiCall();
   console.timeEnd('api-call');
   ```

**Solutions**:
- Add database indexes for frequently queried fields
- Implement caching for repeated requests
- Use connection pooling
- Optimize database queries

### Memory Leaks

**Problem**: Application memory usage keeps increasing.

**Diagnostic Tools**:
```bash
# Monitor memory usage
htop
# or
top -p $(pgrep node)

# Node.js memory profiling
node --inspect your-app.js
```

**Common Causes**:
- Unclosed database connections
- Event listeners not removed
- Large objects not garbage collected
- Circular references

## Environment Issues

### Environment Variables Not Loading

**Problem**: Environment variables are undefined in the application.

**Solutions**:

1. **Check .env file location** (should be in project root)

2. **Verify .env file format**:
   ```bash
   # Correct
   API_KEY=your_key_here
   DATABASE_URL=postgresql://user:pass@host:port/db
   
   # Incorrect (no spaces around =)
   API_KEY = your_key_here
   DATABASE_URL = postgresql://user:pass@host:port/db
   ```

3. **Ensure dotenv is loaded early**:
   ```javascript
   // At the very top of your main file
   require('dotenv').config();
   ```

4. **Check for typos in variable names**:
   ```javascript
   // Make sure these match exactly
   const apiKey = process.env.API_KEY; // not process.env.API_key
   ```

### Port Already in Use

**Problem**: Cannot start server because port is already in use.

**Solutions**:

1. **Find what's using the port**:
   ```bash
   # Linux/macOS
   lsof -i :3000
   
   # Windows
   netstat -ano | findstr :3000
   ```

2. **Kill the process**:
   ```bash
   # Linux/macOS
   kill -9 $(lsof -t -i:3000)
   
   # Windows
   taskkill /PID <PID> /F
   ```

3. **Use a different port**:
   ```bash
   PORT=3001 npm start
   ```

## Getting Help

If you're still experiencing issues:

1. **Check our FAQ**: [FAQ Page]({{ site.baseurl }}/docs/faq/)
2. **Search existing issues**: [GitHub Issues](https://github.com/himaratsu/pagescms-sample/issues)
3. **Create a new issue** with:
   - Detailed description of the problem
   - Steps to reproduce
   - Environment information (OS, Node.js version, etc.)
   - Error messages and logs

### Gathering Debug Information

Before reporting an issue, gather this information:

```bash
# System information
node --version
npm --version
cat /etc/os-release  # Linux
sw_vers             # macOS

# Application logs
tail -f logs/app.log

# Environment variables (remove sensitive data)
env | grep -E "(NODE_|API_|DATABASE_)" | sed 's/=.*/=***/'
```