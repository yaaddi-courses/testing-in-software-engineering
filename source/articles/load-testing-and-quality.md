# Load testing, CPU limits and quality assurance

**Covers:** load test, CPU, QA

## Correct is not enough {#overview}
A program can give the right answer to one user and still fall apart when a thousand arrive together. Whether it holds up depends on its resources, mostly the processor's time. This article explains how a load test shows the limits before real users do, why the processor matters and where this fits into the broader work of checking quality.

## CPU {#cpu}
The CPU, or central processing unit, is the processor that executes a program's instructions. Every request your server handles uses some CPU time. When many requests arrive, they share the processors, and when demand is higher than the CPU can supply, requests wait in a queue and the response time grows.

A useful rule is simple arithmetic: if one request needs 20 milliseconds of CPU time, one core can handle at most 50 requests a second. Four cores can handle about 200. Real systems also wait for disks, databases and networks, so the true limit is usually lower, which is why you measure instead of only calculating.

## load test {#load-test}
A load test simulates many users using a system at once, to see how it behaves under pressure and where it breaks, before real traffic finds out. A tool sends requests at a chosen rate, perhaps starting at 50 per second and rising, while you record the response times, the error rate and the use of CPU and memory.

Variants include a stress test, which pushes beyond the expected limit to see how the system fails, and a soak test, which runs for hours to find slow leaks. The results give you numbers to plan with: how many users the system supports, which part gives out first and what to scale.

Look at the high percentiles of response time, such as the slowest 1 in 100 requests, not just the average. The average hides the worst experiences.

## QA {#qa}
QA, or quality assurance, is the practice of checking that software meets its requirements before it is released. In some companies it is a team of testers, and in others it is a responsibility shared by everyone who builds the software. It covers more than running tests: reviewing requirements, planning what to test, exploring the product by hand to find surprises and tracking defects.

Load tests are one tool in a QA plan, next to unit tests, security checks and manual exploration. The aim is the same: find problems while they are cheap to fix.

## A worked example {#example}
A service uses 20 milliseconds of CPU per request and runs on 4 cores. A QA engineer plans a load test at four traffic levels.

```python
cpu_ms_per_request = 20
cores = 4

max_rps = cores * 1000 / cpu_ms_per_request
print(max_rps)
# 200.0

for users_rps in (100, 150, 200, 250):
    load = users_rps / max_rps
    print(users_rps, f"{load:.0%}")
# 100 50%
# 150 75%
# 200 100%
# 250 125%
```

How to read it:

1. The maximum is 4 cores times 1,000 milliseconds, divided by 20 milliseconds per request: 200 requests per second.
2. At 100 requests per second, the CPU is half used, and response times should stay low.
3. At 150, it is 75 percent used. Queues start to form at the busy moments.
4. At 200, there is no spare capacity. At 250, demand is 125 percent of capacity, so requests pile up and response times climb without limit.

The test confirms where the real limit lies. If the system started to slow at 120 requests per second, well below the calculated 200, that would show another bottleneck, such as the database, worth investigating.

## A common mistake {#mistake}
Testing on a machine that is much smaller or larger than production, so the numbers do not transfer. Run load tests on a setup that resembles the real one. Another mistake is looking only at average response time. A system with a 100 millisecond average can still make 5 percent of users wait 3 seconds.

## Key points {#key-points}
- The CPU is the shared resource that requests compete for. Capacity is cores times time divided by CPU time per request.
- A load test shows how a system behaves under many users and where it breaks.
- Check high percentiles, not only averages.
- QA covers the whole effort to find problems before release, and load testing is one part of it.
