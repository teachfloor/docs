# Rate Limiting
To ensure the stability and performance of the Teachfloor platform, the Teachfloor API enforces a rate limit of **50 requests per minute** per API Key. This limit is designed to prevent abuse and ensure fair usage of resources across all users of the API.

Each HTTP response from the Teachfloor API includes headers that provide real-time information about your current rate limit status. These headers allow you to monitor your usage and adjust your request rate accordingly to avoid hitting the limit.

- `X-RateLimit-Limit` : This header specifies the maximum number of requests allowed per minute. For the Teachfloor API, this value is set to 50.
- `X-RateLimit-Remaining` : This header indicates the number of requests remaining in the current minute. As you make API calls, this number decreases. Once it reaches zero, any further requests within the same minute will be rejected until the rate limit resets.

If you exceed the rate limit, the API will respond with a `429 Too Many Requests` status code. When this happens, it’s important to throttle your requests and wait until the rate limit resets. To help manage this, you can implement logic in your application to monitor the `X-RateLimit-Remaining` header and delay further requests if the limit is nearing exhaustion.