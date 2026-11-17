// Collects headlines from public RSS/Atom feeds into stories.json.
// Only the headline, a short snippet, an optional thumbnail and the link are kept.
import fs from "node:fs";
import crypto from "node:crypto";
import { XMLParser } from "fast-xml-parser";

const CFG = JSON.parse(fs.readFileSync("sources.json", "utf8"));
const OUT = "stories.json";
const FIXTURES = process.env.FEED_FIXTURES || "";
const UA = "Mozilla/5.0 (compatible; 9jatunes-feed-bot/1.0)";
const parser = new XMLParser({ ignoreAttributes: false, attributeNamePrefix: "@_", textNodeName: "#text", processEntities: true });

/* ---------- helpers ---------- */
const arr = (v) => (v == null ? [] : Array.isArray(v) ? v : [v]);
const txt = (v) => {
  if (v == null) return "";
  if (typeof v === "object") return txt(v["#text"] ?? "");
  return String(v);
};
const NAMED = { amp: "&", lt: "<", gt: ">", quot: '"', apos: "'", nbsp: " ", rsquo: "’", lsquo: "‘", rdquo: "”", ldquo: "“", hellip: "…", ndash: "–", mdash: "—" };
const decode = (s) =>
  s
    .replace(/&#(\d+);/g, (_, n) => String.fromCodePoint(+n))
    .replace(/&#x([0-9a-f]+);/gi, (_, n) => String.fromCodePoint(parseInt(n, 16)))
    .replace(/&([a-z]+);/gi, (m, n) => NAMED[n.toLowerCase()] ?? m);
const strip = (h) =>
  decode(
    String(h || "")
      .replace(/<script[\s\S]*?<\/script>/gi, " ")
      .replace(/<style[\s\S]*?<\/style>/gi, " ")
      .replace(/<[^>]+>/g, " ")
  )
    .replace(/The post .{0,200}? appeared first on .{0,80}$/i, "")
    .replace(/\s*(\[…\]|\[\.\.\.\]|Continue reading.*|Read more.*)$/i, "")
    .replace(/[<>]/g, "")
    .replace(/\s+/g, " ")
    .trim();
const clip = (s, n) => {
  if (s.length <= n) return s;
  const cut = s.slice(0, n).replace(/\s+\S*$/, "");
  return cut.replace(/[,.;:\s]+$/, "") + "…";
};
const slug = (s) => s.toLowerCase().replace(/[^a-z0-9]+/g, "-").replace(/^-|-$/g, "");
const sha = (s) => crypto.createHash("sha1").update(s).digest("hex").slice(0, 10);
const norm = (s) => s.toLowerCase().replace(/[^a-z0-9]+/g, " ").trim();

/* ---------- classification ---------- */
const CELEBS = [
  ["Asake", /\bAsake\b/],
  ["Burna Boy", /\bBurna\s?Boy\b/i],
  ["Seyi Vibez", /\bSeyi\s+Vibe[sz]\b/i],
  ["Tiwa Savage", /\bTiwa\s+Savage\b/i],
  ["Tems", /\bTems\b/],
  ["Wizkid", /\bWizkid\b/i],
  ["Davido", /\bDavido\b/i],
  ["Ayra Starr", /\bAyra\s+Starr\b/i],
];
const OTHER_STARS = /\b(Rema|Omah Lay|Fireboy|Olamide|Don Jazzy|Simi|Adekunle Gold|Ckay|Shallipopi|Portable|Zlatan|Phyno|Flavour|2Baba|Yemi Alade|Mr Eazi|Funke Akindele|Genevieve|Mercy Johnson|Kunle Afolayan|Regina Daniels|Toyin Abraham)\b/;
const MOVIES = /\b(nollywood|movie|film|box office|cinema|actor|actress|series|netflix|prime video|amvca|premiere|trailer|skit|comedian)\b/i;
const MUSIC = /\b(album|single|song|ep|music video|concert|tour|afrobeats?|grammy|grammys|headies|chart|streaming|lyrics|rapper|singer|musician|artiste|record label|dj)\b/i;
const ENTERTAINMENT = new RegExp(MOVIES.source + "|" + MUSIC.source + "|\\b(celebrity|celebrities|bbnaija|showbiz|entertainment|influencer|red carpet|wedding|couple)\\b", "i");
const EXCLUDE = /\b(tinubu|apc|pdp|inec|senate|governorship|lawmaker|kidnap|bandits|terrorists)\b/i;
const SPORTS = new RegExp(CFG.sports || "super eagles", "i");

function classify(title, summary, source) {
  const text = `${title} ${summary}`;
  const who = CELEBS.filter(([, re]) => re.test(text)).map(([n]) => n);
  let cat = source.defaultCat || "Gist";
  if (!source.defaultCat) {
    if (who.length || OTHER_STARS.test(text)) cat = "Celebrities";
    else if (MOVIES.test(text)) cat = "Movies";
    else if (MUSIC.test(text)) cat = "Music";
  }
  return { cat, who };
}

/* ---------- feed parsing ---------- */
function firstImage(item, html) {
  const media = [...arr(item["media:content"]), ...arr(item["media:thumbnail"]), ...arr(item["media:group"]?.["media:content"])];
  for (const m of media) {
    const u = m?.["@_url"];
    if (u && (/image/.test(m["@_medium"] || "image") || /\.(jpe?g|png|webp)/i.test(u))) return u;
  }
  for (const e of arr(item.enclosure)) {
    if (e?.["@_url"] && /image/.test(e["@_type"] || "")) return e["@_url"];
  }
  const m = /<img[^>]+src=["']([^"']+)["']/i.exec(html || "");
  return m ? decode(m[1]) : "";
}
function linkOf(item) {
  const l = item.link;
  if (typeof l === "string") return l.trim();
  const list = arr(l);
  const alt = list.find((x) => x?.["@_rel"] === "alternate") || list[0];
  return txt(alt?.["@_href"] || alt).trim();
}

export function parseFeed(xml, source) {
  const doc = parser.parse(xml);
  const items = arr(doc?.rss?.channel?.item).length ? arr(doc.rss.channel.item) : arr(doc?.feed?.entry);
  const out = [];
  const now = Date.now();
  for (const it of items) {
    const title = clip(strip(txt(it.title)), 170);
    const url = linkOf(it);
    if (!title || !/^https?:\/\//.test(url)) continue;
    const rawHtml = txt(it["content:encoded"]) || txt(it.content) || txt(it.description) || txt(it.summary);
    const summary = clip(strip(txt(it.description) || txt(it.summary) || rawHtml), 210);
    let t = new Date(txt(it.pubDate) || txt(it.published) || txt(it.updated) || txt(it["dc:date"])).getTime();
    if (!Number.isFinite(t)) continue;
    if (t > now + 3600e3) t = now;
    const text = `${title} ${summary}`;
    if (source.requireEntertainment && !ENTERTAINMENT.test(text) && !CELEBS.some(([, re]) => re.test(text))) continue;
    if (source.requireSports && !SPORTS.test(text)) continue;
    if (EXCLUDE.test(title)) continue;
    const { cat, who } = classify(title, summary, source);
    let img = CFG.useSourceImages ? firstImage(it, rawHtml) : "";
    if (img.startsWith("http://")) img = "https://" + img.slice(7);
    if (!/^https:\/\//.test(img)) img = "";
    out.push({
      id: "a-" + sha(url),
      auto: true,
      cat,
      who,
      title,
      dek: summary.length >= 40 ? summary : "",
      ts: new Date(t).toISOString(),
      img,
      src: { n: source.name, u: url },
    });
  }
  return out;
}

async function getXml(source) {
  if (FIXTURES) return fs.readFileSync(`${FIXTURES}/${slug(source.name)}.xml`, "utf8");
  const ctl = new AbortController();
  const timer = setTimeout(() => ctl.abort(), 20000);
  try {
    const res = await fetch(source.url, { headers: { "User-Agent": UA, Accept: "application/rss+xml, application/xml, text/xml, */*" }, signal: ctl.signal, redirect: "follow" });
    if (!res.ok) throw new Error("HTTP " + res.status);
    return await res.text();
  } finally {
    clearTimeout(timer);
  }
}

/* ---------- main ---------- */
const status = [];
let fresh = [];
for (const source of CFG.sources) {
  try {
    const xml = await getXml(source);
    const items = parseFeed(xml, source).sort((a, b) => b.ts.localeCompare(a.ts)).slice(0, CFG.perSourceLimit || 12);
    fresh = fresh.concat(items);
    status.push({ name: source.name, ok: true, items: items.length });
    console.log(`OK    ${source.name}: ${items.length} stories`);
  } catch (err) {
    status.push({ name: source.name, ok: false, items: 0, error: String(err.message || err).slice(0, 120) });
    console.log(`SKIP  ${source.name}: ${err.message || err}`);
  }
}

let previous = [];
try {
  previous = JSON.parse(fs.readFileSync(OUT, "utf8")).stories || [];
} catch {}

const cutoff = Date.now() - (CFG.maxAgeDays || 21) * 86400e3;
const byId = new Map();
for (const s of [...previous, ...fresh]) byId.set(s.id, s);
const seenTitles = new Set();
const stories = [...byId.values()]
  .filter((s) => new Date(s.ts).getTime() >= cutoff)
  .sort((a, b) => b.ts.localeCompare(a.ts))
  .filter((s) => {
    const k = norm(s.title);
    if (seenTitles.has(k)) return false;
    seenTitles.add(k);
    return true;
  })
  .slice(0, CFG.maxStories || 80);

fs.writeFileSync(OUT, JSON.stringify({ updated: new Date().toISOString(), sources: status, stories }, null, 1) + "\n");
console.log(`Wrote ${stories.length} stories (${status.filter((s) => s.ok).length}/${status.length} sources responded).`);
