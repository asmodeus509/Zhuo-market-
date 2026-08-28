# Zhuo Market Backend on Vercel

## Important
This version keeps the existing SQLite database code for local development. SQLite files are not persistent on Vercel serverless deployments, so production data can disappear between executions.

For a real production deployment, migrate `src/db.js` to a hosted database such as Neon Postgres or Supabase.

## Deploy
1. Upload this folder to GitHub.
2. Import the repository in Vercel.
3. Framework preset: Other.
4. Add environment variables: JWT_SECRET, FRONTEND_ORIGIN.
5. Deploy.

Health check: /api/health
