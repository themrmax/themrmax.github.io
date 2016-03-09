---
title: Learning to think in vectors
layout: post
---

One of the things I really struggled with when I started learning `pandas` was to think in terms of vectorised operations instead of loops and iterations. At the time I remember wishing there were some exercises I could work through to get a jump on this, so I've decided to go through some of my early Pandas code, and see if I can offer 2-years-ago me that help I was asking for (hopefully this will help someone else too!)


### Convert a column to integer

Don't: `df.iloc[0,:] = [int(x) for x in df.iloc[0,:]]`
Do: `df.iloc[0,:] = df.iloc[0,:].astype(int)`

## Compute a column to cumlative percentage
Don't:

    def accumu(lis):
        total = 0
        for x in lis:
            total += x
            yield total
        N = sum(V)
        W = [float(w)/N for w in list(accumu(V[0:4000]))]

Do: `W = V.cumsum()` 