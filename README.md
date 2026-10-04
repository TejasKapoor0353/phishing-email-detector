# Phishing Email Pattern Detector

A phishing-email detector built from transparent rules. **No machine learning**: each email gets a score from weighted indicators, and a score of 5 or more is flagged as phishing. Pure Python standard library, so there is nothing to install.

## Run locally

```bash
python server.py        # website at http://127.0.0.1:8000
python demo.py          # same demonstration in the terminal
python -m unittest discover -s tests -v
```

Requires Python 3.9+. The server binds to `127.0.0.1` only; emails never leave your machine.

## Demonstration

`phishing_detector/samples.py` holds 5 phishing and 5 legitimate emails (one legitimate bank alert deliberately uses urgent wording). The website and `demo.py` show each verdict, whether it was correct, the false positives and false negatives, and every indicator that fired.

| Result | Count |
|---|---|
| Correct | 10 / 10 |
| False positives | 0 |
| False negatives | 0 |

The first run produced **one false positive** (the bank alert). Causes: substring matching ("b**locked**" matched "locked") and a "never share your OTP" warning counted as a credential request. Both were fixed (whole-word matching, warning-context filter) and the fixes have unit tests. The refinement happened after seeing the demo set, so 10/10 is **not** a real accuracy estimate.

## Indicators

| Indicator | Points |
|---|---|
| Look-alike sender or link domain (`paypa1-secure.com`) | 4 |
| Link text domain differs from the real href | 4 |
| Link to a raw IP address | 4 |
| Risky attachment type (`.zip .exe .html .docm ...`) | 4 |
| Double-extension attachment (`label.pdf.zip`) | 3 |
| Display name claims a brand, sender domain does not match | 3 |
| Requests passwords, card numbers, OTP, identity details | 3 |
| Punycode domain, `@` inside a URL | 3 |
| Urgency wording | 2-3 |
| Threat wording | 2 |
| Lure / payment scam wording (lottery, gift cards, wire transfer) | 2 |
| Reply-To differs from sender; brand mentioned from unrelated domain; URL shortener | 2 |
| Generic greeting; many subdomains; plain `http`; ALL-CAPS/exclamations | 1 |

Tune `THRESHOLD`, the weights, and the word lists at the top of `phishing_detector/detector.py`.

## Limitations

- Tiny test set; fixed word lists miss new wording and non-English text.
- No SPF/DKIM/DMARC checks, domain-age lookup, or link reputation.
- The look-alike check only knows the brands listed in `BRANDS`.

## Layout

```
phishing_detector/   detector.py (rules), samples.py (10 labelled emails)
web/index.html       the website
server.py            local server + JSON API (/api/demo, /api/analyze)
demo.py              terminal demo
tests/               unit tests
.github/workflows/   CI
```

## Publish to GitHub

```bash
git init -b main && git add . && git commit -m "Initial commit"
# create an empty repo on github.com, then:
git remote add origin https://github.com/<you>/phishing-email-detector.git
git push -u origin main
```

MIT licensed.
