const http = require("http");
const fs = require("fs");
const path = require("path");
const url = require("url");

const PORT = 8080;
const PUBLIC = path.join(__dirname, "public");
const MEDIA = path.join(PUBLIC, "media");

const mime = {
  ".html": "text/html; charset=utf-8",
  ".jpg": "image/jpeg",
  ".jpeg": "image/jpeg",
  ".png": "image/png",
  ".gif": "image/gif",
  ".webp": "image/webp",
  ".mp4": "video/mp4",
  ".mov": "video/quicktime",
  ".mkv": "video/x-matroska"
};

const server = http.createServer((req, res) => {
  const pathname = decodeURIComponent(url.parse(req.url).pathname);

  // Media file list
  if (pathname === "/media/" || pathname === "/media") {
    fs.readdir(MEDIA, { withFileTypes: true }, (err, entries) => {
      if (err) {
        res.writeHead(500, { "Content-Type": "application/json" });
        return res.end(JSON.stringify({ error: err.message }));
      }

      const files = entries
        .filter(e => e.isFile())
        .map(e => e.name);

      res.writeHead(200, {
        "Content-Type": "application/json; charset=utf-8",
        "Access-Control-Allow-Origin": "*"
      });

      res.end(JSON.stringify(files));
    });
    return;
  }

  // Static files
  let filePath;

  if (pathname === "/") {
    filePath = path.join(PUBLIC, "index.html");
  } else {
    filePath = path.join(PUBLIC, pathname);
  }

  // Security: keep requests inside public
  if (!filePath.startsWith(PUBLIC)) {
    res.writeHead(403);
    return res.end("Forbidden");
  }

  fs.stat(filePath, (err, stat) => {
    if (err || !stat.isFile()) {
      res.writeHead(404);
      return res.end("Not Found");
    }

    const ext = path.extname(filePath).toLowerCase();

    res.writeHead(200, {
      "Content-Type": mime[ext] || "application/octet-stream"
    });

    fs.createReadStream(filePath).pipe(res);
  });
});

server.listen(PORT, "0.0.0.0", () => {
  console.log(`Gallery running at http://127.0.0.1:${PORT}`);
});
