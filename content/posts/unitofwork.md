---
title: "unit of work實作"
date: 2022-10-20T16:52:04+08:00
draft: false
tags: ["Csharp"]
type: post
author: "Steve Lin"
description: ""
---

# unit of work實作
```C#
 public interface IUnitOfWork : IDisposable
    {
        IDbConnection Connection { get; }

        IDbTransaction Transaction { get; }

        void BeginTransaction();

        void Commit();

        void RollBack();
    }
```

```C#
public class UnitOfWork : IUnitOfWork
    {
        public IDbConnection Connection { get; private set; }

        public IDbTransaction Transaction { get; private set; }

        public Guid _id = Guid.Empty;

        public UnitOfWork(IDbConnection connection)
        {
            _id = Guid.NewGuid();
            Connection = connection;
            DefaultTypeMap.MatchNamesWithUnderscores = true;
        }

        public void BeginTransaction()
        {
            if (!Connection.State.Equals(ConnectionState.Open))
            {
                Connection.Open();
            }

            if (this.Transaction != null)
                return;

            this.Transaction = Connection.BeginTransaction();
        }

        public void Commit()
        {
            this.Transaction.Commit();

            if (this.Transaction != null)
            {
                this.Transaction.Dispose();

                this.Transaction = null;
            }
        }

        public void Dispose()
        {
            if (this.Transaction != null)
            {
                this.Transaction.Dispose();
            }
                
            this.Transaction = null;
        }

        public void RollBack()
        {
            this.Transaction?.Rollback();

            if (this.Transaction != null)
            {
                this.Transaction.Dispose();

                this.Transaction = null;
            }
        }
    }
```