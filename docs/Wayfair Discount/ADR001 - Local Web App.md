# ADR001 Local Web App MVP
The goal for this project is to automate receiving discount requests from friends/family, checking it for them, and returning the relevant results with as little interruption to me as possible.

## Status

Complete

## Context

Providing Wayfair discounts to friends and family is allowed and encouraged, but the discount is not a fixed amount and varies by wholesale cost, and therefore varies by product, shipping location, and time that it is checked. To perform a discount check for someone you need to either find the wholesale cost and apply the calculation, or manually add the product to your cart, and apply the employee discount check. Therefore, it's tedious to check discounts for people in a timely manner and without them feeling like they are bothering me. 

### Limitations

* Getting wholesale price requires VPN access so it cannot be done in a hands-off automated way.
* Password sharing is not an option.
* I have friends in both US and Canada so will have to support wayfair.com and wayfair.ca
* Getting discount info requires the product to be in your Wayfair cart and manually applying the employee discount to the cart.

## Decision

Create a Next.js web application that will host a very basic input form for users to provide their zip/postal code and a link to a wayfair (.com or .ca) product page. Submitting the form will send the request to the backend server to be queued up, discount checked, and results saved for future reference for the user.

To do the actual discount fetching, a Puppeteer NodeJS script will be used 

MVP Architecture

<iframe src="https://viewer.diagrams.net/?tags=%7B%7D&highlight=0000ff&edit=_blank&layers=1&nav=1&title=WayfairDiscountMvp.drawio#R3VnZkto4FP0aqpKHprxglscGmiyVpKimZnryNCVsYatbthxZZsnXz5UtL4pommQIkH6hpOur7Zx7jxY67iTevuMojT6zANOOYwXbjjvtOM7IGsGvNOxKgzfsl4aQk6A02Y1hQb5jZbSUNScBzjRHwRgVJNWNPksS7AvNhjhnG91txag%2BaopCbBgWPqKm9YEEIiqtQ89q7O8xCaNqZNtSX2JUOStDFqGAbVom967jTjhjoizF2wmmErsKl7Ld7Jmv9cQ4TsQxDaL7cP1g%2Fx0ky6fpv735aP714%2FTGHpTdrBHN1YrfsxgvMF9jDvY3n5G%2FZOwJireEv1UrEbsKHvASBND6hJaYzllGBGEJfFoyIVjccceVwy0lofwgWArWSMQUKjYUAZZUdhZvQxlA3SXKiN%2FlBZfjFaF0wijjgNY0YQmWDQRnTzUZRRdl1MD63TFJghIPr928aR0xTr6zRKBqfLV%2BmCbePousXfMFcY4BIMF34KIauD1FsYpxu6pvmojpVXERtaLFrWIDqSgN674bIqGguPwZXi2DV4M7HECkqyrjImIhSxC9a6xjznKAU44joW18PjHJYgHfIxZip9IW5YLp5JZjyoEOQwvzYjn38YEVqfkLxEMsDviN9lPFMUWCrPV5nB5214D9Hn%2FLcSb2ol9kjY4YUnniA0CQgWYCxSQISnIwxD1aFv1JelJGElEsyBt3vOmh0FZiqRo3EtXm5UBYPZsIN1bX8byRlgyKt6PxV53P5WpaLmy1yrAwCKrn8OucOQZlM05wEkBnMxQTujOoqyQrj%2BmtL1ibpV%2FXQZYLShI8qfcx6zTiVG9Iig%2FXNsXJ3aNN%2Fd8mTSbgf7g0jY6UJvsZps6jTc7olWuTczAPrK7V7w%2F0ffrqpek6dnHAne%2F%2Bke27XlX92v423arOy9pOU6%2Fzptjwoim2b%2FtHsJNY9zjLKWTAq8g058VTgDVye1qq3Vx9rpnqOCWZD6kjh%2F%2FrQ0cueiavQXMo96m8nQRkDcVQFt98wVvRfcykQ5q%2BrTygScvJYH8TEYEXKSpCfwPXHz0K9p8ZTnAmcEfei2cC2znnoaAWgqsQNut8wmZXjxpXrmzmoe2BQ%2FC%2BOmk7nDjygtOzhlryXP0FpwqxFnfzPE2xwMUTz8LnJBWVYi15I2kswKWkZYWL%2BfpzLfplDy6tX%2FbQwHhGqEyPBVwQ5Rujie9jxhITU13UXkD4BFh6A%2F1%2B6Ox5vDrzXjC48F4w%2BKlTLt4SUbZzPFWtm0G5aSUrFzwa%2F9%2BHsaLpLedo13JQmmzKmoqunqNnau%2FHR%2Bof%2FYcH%2FaFQzuC01ywzeR8QkbNaMV7L355TH8SQYy1llqM1IrTYbo47%2Bf2Zm%2BPgiM1R5696XbvE7gjV5t%2BV0r35i8q9%2Bw8%3D" title="MVP Architecture" width="800" height="400"></iframe>

## Consequences

* The script can only be run one at a time, so a semaphore has to be used to restrict multiple requests from being executed simultanesously.
* The api waits for the puppeteer script to finish, so it can result in a long user wait (especially if simultaneous requests are being processed).
* Home server solution is scrappy, but would be better to host it in a free hosting environment like Vercel.
* The puppeteer solution cannot be used off-prem due to security and botting detection reasons.
* It's clunky to have to open the web app, paste a link, and have to wait for the results.