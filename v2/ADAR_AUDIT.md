# Adar audit

Reference: `@hebcal/core` 5.3.0, the same pinned build both pages load from jsDelivr (`cdn.jsdelivr.net/npm/@hebcal/core@5.3.0/dist/bundle.min.js`). The eight anchors below also match `Intl.DateTimeFormat('en-US-u-ca-hebrew')` on the Gregorian day Hebcal returns. There is no second calendar and no hand-written month arithmetic. The page walks `HDate.next()` / `HDate.prev()`.

The day list on the live page always has 30 options. Hebcal does not reject a day the month does not have. `new HDate(30, 'Adar II', 5787)` becomes 1 Nisan 5787. `new HDate(30, 'Adar', 5788)` becomes 1 Nisan 5788. `new HDate(30, 'Cheshvan', 5784)` becomes 1 Kislev 5784. The chart is then 355 real, consecutive days starting from that rolled date. Valid dates, including 1–29 Adar II and 30 Adar I, are printed correctly. The bug is the anchor, not a scrambled Adar II column.

v2 asks Hebcal how many days the selected month has and only offers those days. Adar I has 30. Adar II and non-leap Adar have 29. Cheshvan and Kislev follow the year. Day 30 of a 29-day month is not in the list, so it cannot roll into the next month.

The artwork is still 12 columns and 355 days. A leap year has 13 months. A 355-day range that crosses a leap year shows both אדר א׳ and אדר ב׳ in the headers. It does not become a 13-month chart.

## Targeted check

Each chart was generated through the real form, reloaded between cases. All 355 day cells were read in reading order and compared with the reference, including the month header above each column. One PDF per page (`29 Adar II 5787`) was printed and its text was checked for all 355 date strings.

| Case | Original | v2 |
| --- | --- | --- |
| End, 30 Adar I 5787 | 355/355, header ל׳ אדר א׳ תשפ״ז | 355/355, same |
| End, 29 Adar II 5787 | 355/355, header כ״ט אדר ב׳ תשפ״ז. PDF 355/355 | 355/355, same. PDF 355/355 |
| End, 29 Adar 5788 (not a leap year) | 355/355, כ״ט אדר תשפ״ח | 355/355, same |
| End, 30 Adar 5788 | Offered. Prints א׳ ניסן תשפ״ח. 355 cells then follow that rolled date | Day 30 is not offered (list length 29) |
| End, 1 Nisan 5787 | 355/355, א׳ ניסן תשפ״ז | 355/355, same |
| Start, 15 Shvat 5786, crossing into leap 5787 | 355/355, ends י״ד שבט תשפ״ז. Headers match, including both Adars | 355/355, same |
| End, 29 Kislev 5787 | 355/355, כ״ט כסלו תשפ״ז | 355/355, same |
| End, 30 Cheshvan 5784 | Offered. Prints א׳ כסלו תשפ״ד | Day 30 is not offered (list length 29) |

Valid charts: 6 on the original and 6 on v2, 2,130 cells each, 0 mismatches, 0 broken month steps, 0 day-30 in a 29-day month, 0 header mismatches. PDF text: 355/355 on each of the two printed charts, 0 missing strings.

## Ink centering

The slot box was already centered. The letters sat low inside it because of the font's ascent and descent. Each string is now shifted by the measured ink: `0.35 * fontSize + (descent - ascent) / 2`, upward. The download canvas uses the same ink bounds.

On a white cell, ink center versus slot center:

| Slot | Before | After |
| --- | --- | --- |
| Date | +0.13px | +0.06px |
| Month header | +1.25px | −0.25px |
| Date range | +0.25px | −0.13px |
| Dedication | +2.06px | +0.19px |

After the change, every measured ink center is within 0.5px of the slot center.
