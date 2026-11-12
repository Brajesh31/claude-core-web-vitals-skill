# Largest Contentful Paint (LCP)

## What LCP measures

Largest Contentful Paint measures when the largest image, video poster, or text block in the visible viewport is rendered. It is a loading-experience metric. The identity of the LCP element can vary by viewport, content, and page state.

Use field data to understand real visitors. As a practical target, aim for LCP at or below 2.5 seconds for at least 75% of visits. Above 4 seconds is poor. Analyze mobile and desktop separately where traffic and layout differ.

## The four parts of LCP timing

A useful diagnostic model separates LCP into:

1. **Time to first byte (TTFB):** time until the first response byte.
2. **Resource load delay:** time between response start and discovery/request of the LCP resource.
3. **Resource load duration:** time spent downloading the LCP resource.
4. **Element render delay:** time between resource completion and the element being painted.

Identify which part dominates before changing code. A smaller image does not solve an LCP problem caused by late discovery or blocked rendering.

## Find the actual LCP element

- Inspect a representative mobile and desktop page.
- Use browser performance tooling and field diagnostics to identify the element.
- Check whether the element changes after hydration, personalization, or font loading.
- Inspect the request waterfall for the resource and its initiator.
- Confirm that the measured element is the intended primary content, not a transient placeholder.

A lightweight observer can help during local debugging:

~~~js
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log("LCP candidate", {
      startTime: entry.startTime,
      element: entry.element,
      url: entry.url,
      size: entry.size,
    });
  }
});

observer.observe({ type: "largest-contentful-paint", buffered: true });
~~~

This is a diagnostic aid. Production measurement should use a maintained Web Vitals implementation or browser tooling and account for page lifecycle behavior.

## Reduce each timing component

### 1. Improve TTFB

- Use caching for stable, reusable responses.
- Reduce slow database queries and server-side work on the critical request path.
- Use a CDN or edge delivery where it benefits the audience.
- Avoid redirects before the document response.
- Measure origin response time separately from network and cache behavior.

A low TTFB is helpful but does not guarantee a good LCP.

### 2. Discover the LCP resource early

- Make the main image discoverable from the initial HTML when possible.
- Avoid injecting the LCP image only after client-side JavaScript runs.
- Do not lazy-load the likely LCP image.
- Consider preloading only when the resource would otherwise be discovered late.
- Use a high fetch priority for the primary above-the-fold image when appropriate.
- Check that CSS background images are not hiding the resource from early discovery.

Example:

~~~html
<img
  src="/images/hero.webp"
  width="1440"
  height="900"
  alt="Description of the hero image"
  fetchpriority="high"
/>
~~~

Do not mark every image as high priority. Reserve priority for the resource that actually controls the user's initial experience.

### 3. Reduce resource load duration

- Serve the correct responsive image size.
- Use modern image formats when supported and beneficial.
- Compress images without visible quality loss.
- Avoid shipping unnecessarily large video or image assets.
- Use an appropriate cache policy for versioned assets.
- Check transfer size and request priority in the network waterfall.

Do not preload multiple competing hero variants. Ensure responsive preload rules match the image the browser will actually request.

### 4. Reduce element render delay

- Keep critical CSS small and available early.
- Remove or defer non-critical render-blocking scripts.
- Avoid long main-thread tasks during initial rendering.
- Ensure required fonts load efficiently and have a sensible fallback.
- Minimize expensive layout, style, and hydration work before the hero can paint.
- Prefer server-rendered or statically generated primary content when it suits the application.

Do not inline an entire stylesheet to solve one critical rule. Keep critical CSS focused.

## Common failure patterns

| Symptom | Likely cause | Investigation |
|---|---|---|
| High TTFB | Slow origin, uncached HTML, server-side work | Server traces, cache headers, backend timing |
| Image request starts late | CSS or JavaScript discovery, lazy loading | Initial HTML and request initiator |
| Large image takes too long | Oversized asset, poor compression, slow delivery | Transfer size, dimensions, CDN behavior |
| Image loads but paints late | Main-thread work, CSS, font, or layout delays | Performance trace and long tasks |
| Lab score good, field LCP poor | Real devices, network, personalization, geography | Field data segmented by device and route |

## Measurement workflow

1. Record field LCP at the 75th percentile for the affected page group.
2. Segment by device class and relevant page templates.
3. Reproduce the issue in a controlled lab environment.
4. Identify the LCP element and timing component that dominates.
5. Make one focused change.
6. Re-test with the same viewport and conditions.
7. Monitor field data after deployment.

Do not claim a production improvement from a single Lighthouse run. Lab results diagnose; field results show real-user experience.

## Regression checklist

- [ ] The correct LCP element was identified.
- [ ] The LCP asset is not lazy-loaded.
- [ ] The browser discovers the resource early.
- [ ] Image dimensions and format match the rendered size.
- [ ] Priority hints are limited to critical resources.
- [ ] Critical CSS and fonts do not delay the paint.
- [ ] Main-thread work does not block rendering.
- [ ] Mobile and desktop results were inspected.
- [ ] Field data is monitored after release.

## References and cautions

Thresholds and browser measurement details can evolve. Verify current metric definitions and diagnostic recommendations in the official web performance documentation before making a release decision. LCP is one part of user experience and should be improved alongside responsiveness, visual stability, accessibility, and task completion.
