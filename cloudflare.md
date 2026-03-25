/**
 * Welcome to Cloudflare Workers! This is your first worker.
 *
 * - Run "npm run dev" in your terminal to start a development server
 * - Open a browser tab at http://localhost:8787/ to see your worker in action
 * - Run "npm run deploy" to publish your worker
 *
 * Learn more at https://developers.cloudflare.com/workers/
 */

export default {
  async fetch(request) {
    const url = new URL(request.url);

    // 1) 必须是 HTTPS
    const APPLE_BASE = "https://sat-cdn.apple-mapkit.com/tile";

    // 2) 仅允许 GET/OPTIONS（便于 CORS 预检）
    if (request.method === "OPTIONS") {
      return new Response(null, {
        headers: {
          "Access-Control-Allow-Origin": "*",
          "Access-Control-Allow-Headers": "*",
          "Access-Control-Allow-Methods": "GET, OPTIONS",
        }
      });
    }
    if (request.method !== "GET") {
      return new Response("Method Not Allowed", { status: 405 });
    }

    // 3) 直接透传查询参数（style/size/scale/z/x/y/v/accessKey）
    //   注意：前端一定要用 &，不能用 &amp;
    const target = APPLE_BASE + url.search;

    // 4) 增强请求头（更像浏览器）
    const upstream = await fetch(target, {
      headers: {
        "User-Agent": "Mozilla/5.0 TileProxy/1.0",
        "Accept": "image/avif,image/webp,image/apng,image/*,*/*;q=0.8",
        // 有些场景可帮助放行（也可能不需要）
        "Referer": "https://maps.apple.com/"
      }
      // 不用手动设置 Host，平台会根据目标自动带对的 SNI/Host
    });

    // 5) 回传图片 + CORS 头
    return new Response(upstream.body, {
      status: upstream.status,
      headers: {
        "Content-Type": upstream.headers.get("Content-Type") || "image/jpeg",
        "Access-Control-Allow-Origin": "*",
        "Access-Control-Allow-Headers": "*",
        "Access-Control-Allow-Methods": "GET, OPTIONS",
        "Cache-Control": "public, max-age=600"
      }
    });
  }
}
