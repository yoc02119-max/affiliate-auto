name: Rakuten Affiliate Auto Test

"on":
  workflow_dispatch:

jobs:
  rakuten-test:
    runs-on: ubuntu-latest

    steps:
      - name: Call Rakuten Ichiba API
        env:
          RAKUTEN_APPLICATION_ID: ${{ secrets.RAKUTEN_APPLICATION_ID }}
          RAKUTEN_ACCESS_KEY: ${{ secrets.RAKUTEN_ACCESS_KEY }}
          RAKUTEN_AFFILIATE_ID: ${{ secrets.RAKUTEN_AFFILIATE_ID }}
        run: |
          python3 - <<'PY'
          import os
          import json
          import urllib.parse
          import urllib.request

          base = "https://openapi.rakuten.co.jp/ichibams/api/IchibaItem/Search/20260701"

          params = {
              "applicationId": os.environ["RAKUTEN_APPLICATION_ID"],
              "accessKey": os.environ["RAKUTEN_ACCESS_KEY"],
              "affiliateId": os.environ["RAKUTEN_AFFILIATE_ID"],
              "keyword": "楽天",
              "hits": 5,
              "format": "json",
          }

          url = base + "?" + urllib.parse.urlencode(params)

          req = urllib.request.Request(
              url,
              headers={
                  "User-Agent": "Mozilla/5.0",
                  "Accept": "application/json",
              },
          )

          with urllib.request.urlopen(req, timeout=30) as response:
              body = response.read().decode("utf-8")

          data = json.loads(body)
          items = data.get("Items", [])

          print("取得件数:", len(items))

          for i, item_wrap in enumerate(items[:5], 1):
              item = item_wrap.get("Item", item_wrap)
              print("-----", i, "-----")
              print("商品名:", item.get("itemName"))
              print("価格:", item.get("itemPrice"))
              print("Affiliate URL:", item.get("affiliateUrl"))
          PY
