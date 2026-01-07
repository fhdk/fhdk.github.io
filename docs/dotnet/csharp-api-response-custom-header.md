---
title: 'Web API2 Expose Custom Header'
taxonomy:
    category:
        - docs
---

Customize API response with Custom Header and Expose the header to the client.

```
using System.Collections.Generic;
using System.Net.Http;
using System.Net.Http.Formatting;
using System.Threading;
using System.Threading.Tasks;
using System.Web.Http;
using System.Web.Http.Results;

namespace Inno.Api.Results
{
    public class CustomOkResult<T> : OkNegotiatedContentResult<T>
    {
        public CustomOkResult(T content, IContentNegotiator contentNegotiator, HttpRequestMessage request, IEnumerable<MediaTypeFormatter> formatters) : base(content, contentNegotiator, request, formatters)
        {
        }

        public CustomOkResult(T content, ApiController controller) : base(content, controller)
        {
        }

        public string XPaginationContent { get; set; }

        public override async Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
        {
            var response = await base.ExecuteAsync(cancellationToken);
            response.Headers.Add("Access-Control-Expose-Headers", "X-Pagination");
            response.Headers.Add("X-Pagination", XPaginationContent);            
            return response;
        }
    }
}
```

In controller response

```
return new CustomOkResult<List<CrmCompany>>(companies, this)
{
    XPaginationContent = JsonConvert.SerializeObject(companies.MetaData)
};
```