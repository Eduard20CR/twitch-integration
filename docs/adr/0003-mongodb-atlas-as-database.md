# ADR-0003 — MongoDB on Atlas as the database

Status: Accepted

We keep Streamers and Results in MongoDB: each Game stores a Result with its own shape, and a document per Result fits that without a schema per Game. In production it runs on MongoDB Atlas, which is real MongoDB and has a free tier to start; locally it is the official `mongo` image in docker-compose. Because MongoDB has no foreign keys or cascades, the App Server deletes a Streamer's data itself, in steps that can be retried.

## Considered options

- **PostgreSQL (RDS) with a `JSONB` column**: also fits the different Result shapes, and cascades deletes, but the user chose MongoDB.
- **Amazon DocumentDB**: inside AWS, but only compatible with MongoDB, not MongoDB itself, and its smallest instance costs more.
