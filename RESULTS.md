# K6 lab results

## Part 0

- Step 0.1, http_req_duration: avg=min=med=max=p(90)=p(95)=260.81 ms (1 request)
- Step 0.2, requests, rate and p(95), and why the rate stays below 5:
  80 requests, 3.83 req/s, p(95) = 268.82 ms. Each VU sends one request
  (about 0.26 s) and then sleeps for 1 s, so one iteration takes about
  1.29 s (iteration_duration avg). Each VU therefore sends only about
  0.78 requests per second, and 5 VUs send about 3.9 requests per second,
  which is below 5. The sleep (think time) is the reason.
- Step 0.6, exit code of the normal run and of the run with P95=300:
  Normal run: exit code 0 (all thresholds passed, p(95) = 372.89 ms < 1500 ms).
  Run with P95=300: exit code 99 (p(95) = 376.22 ms is not < 300 ms, so the
  http_req_duration threshold was crossed).
- Step 0.7, rules that passed, p(95), p(99), exit code after your change:
  All 3 rules passed: http_req_failed rate = X%, p(95) = X ms (< 1000),
  p(99) = X ms (< 1500). After changing p(95)<1000 to p(95)<100,
  the p(95) rule failed and the exit code was 99.
- Step 0.8, what was wrong, and how the checks showed it:

## Part A results

| Step | Test type    | Users | Req/s | p(95) | Max | Failed | Checks |
| ---- | ------------ | ----- | ----- | ----- | --- | ------ | ------ |
| A1   | Smoke        | 1     |       |       |     |        |        |
| A2   | Average load | 3     |       |       |     |        |        |
| A3   | Stress       | 5     |       |       |     |        |        |
| A4   | Spike        | 5     |       |       |     |        |        |
| A5   | Soak         | 2     |       |       |     |        |        |

## Part B

1.
2.
3.
4.
5.
6.
7.
8.
9.