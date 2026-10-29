# How to get a task done through the interface

If the needed action is missing from the skill's commands or the API, prepare everything a person needs to do it. This applies to creating campaigns, setting up goals, working with assets, and any other manual step. Below is the general procedure and an example for the Campaign Wizard.

## Handing off a manual step

First explain what result is needed and why this setting was chosen. Check whether it can be done with the existing commands. The lack of a ready-made command and the lack of an API capability are different limitations: name the actual reason.

Give the person instructions they can follow without designing an ad campaign themselves:

1. **What to open.** The service, the account, the specific campaign or counter, the path to the section. Check button names against the current help or the interface you can access. If the path has changed, look up the current location; do not invent one.
2. **What to enter.** Prepared texts, links, files, the list of goals with IDs, parameter values. Mark missing data explicitly: "Weekly budget is not set: an amount must be chosen". Do not fill it in with random numbers.
3. **Why it is needed.** One short explanation for any significant setting: which result it improves or which limit it sets.
4. **What happens after the click.** Saving a draft, sending to moderation, changing running ads, or starting to spend money.
5. **How to check the result.** The expected value after saving, an event in Metrica, the campaign status, or a report row. State who performs the check and what information must be returned to the agent.

Prepare the assets and the table of final settings before handing off the manual step. Reuse goals and values already agreed in the conversation. If launching is part of the assignment and the parameters are defined, do not add another approval without a reason. If only preparation was requested, leave the result ready to launch and clearly mark the final action.

After the manual step is done, check the available statistics or the saved settings. Do not call the action done if only the instructions were prepared.

## Example: Campaign Wizard (Мастер кампаний) for promoting a website

This example fits when an automatic campaign is needed to get the target actions on the site that the user has chosen. The skill's current commands do not create such campaigns; statistics after creation can be pulled through Reports. The tool's own capabilities are described in the [Campaign Wizard help](https://yandex.ru/support/direct/ru/campaign-master/about).

### 1. Prepare data and assets

Before setup, collect or prepare:

| What you will need | What must be defined |
|---|---|
| Account and site | Login, full link to the promoted page, name of the future campaign |
| Offer | What is sold, to whom, at what current price, within what time frame and in what territory |
| Ad result | One or more valuable goals chosen by the user |
| Handling of requests | Where the request arrives, who answers and when; for sales, where payment is confirmed |
| Money | Acceptable weekly spend, launch period, acceptable cost of each chosen goal |
| Ads | Ready headlines, texts, images or videos, sitelinks and contacts |

Do not stop at the request "add good ads". Prepare variants for the actual offer. Check month names, prices, promotion dates and terms against the site. For images, provide ready files or an exact brief for the missing asset.

Every text must have a clear offer and a next step for the customer. Links must lead to the matching products, services or terms. Double-check combinations of headlines, text and images: each must keep its meaning and the current price. [Yandex recommendations on headlines and texts](https://yandex.ru/support/direct/ru/efficiency/text-and-title).

### 2. Prepare the counter and access

Open the needed counter in Metrica and check the domain. If there is no counter yet, click **Add counter (Добавить счётчик)**, fill in the site details and install the resulting code on the site's pages. The installation code is available under **Settings (Настройка) → Counter (Счётчик) → Copy (Скопировать)**; give the site owner or developer the specific domain and counter number. [Creating and installing a counter](https://yandex.ru/support/metrica/ru/quick-start).

If Metrica and Direct use different logins, the counter owner opens **Settings (Настройки) → Access (Доступ) → Individual access (Индивидуальный доступ) → Add user (Добавить пользователя)**, enters the needed login, selects the permissions that match the task and saves the changes. View permission is enough for viewing; editing an existing goal requires edit permission. There is no need to give a password or token to a person or an agent for this. [Counter access](https://yandex.ru/support/metrica/ru/general/access).

Result of the step: the counter number, the site, and the login that sees the counter in Direct are known.

### 3. Create or check the valuable goals

Show the user the existing goals with names and IDs. Find out which fact means a business result. Reuse an existing working goal.

| Result | When to count the goal as reached | What is not enough on its own |
|---|---|---|
| Accepted request | The server accepted the form and created a record in the request handling system | A button click, an unfilled form, an error message |
| Confirmed payment | The order received a confirmed payment status | Adding to the cart, going to payment, returning from the payment system without a status check |
| Qualified call | The call tracking system confirmed the agreed signs of a target call | Tapping the phone number |

If a new goal is needed, in Metrica open **Goals (Цели) → Add goal (Добавить цель) → Target event (Целевое событие)**. Set a clear name and an event identifier, for example `lead_accepted` for an accepted request. Choose exact match for the identifier and save the goal. The site sends the event through `reachGoal`; for events outside the site, offline data upload is used. [Setting up a target event](https://yandex.ru/support/metrica/ru/general/goal-js-event).

Give the site developer a specific task:

> For the counter with the given number, send `reachGoal` with the identifier `lead_accepted` after the request is successfully created. Do not send it on a field validation error, a server rejection or a plain click. Re-rendering the success message must not create a repeat goal for the same request. For payment, create a separate event after payment is confirmed, if the user chose exactly that result.

The event names in this example are suggested names for new goals. For an existing goal, keep its real event name. The numeric goal ID for Direct and the text event identifier for the site are different values; record both.

If confirmed payment is known only to the order accounting system, set up the transfer from there. In Metrica, data from a CRM can create separate goals, including a paid order. Choose exactly the status you need. [Uploading data from a CRM](https://yandex.ru/support/metrica/ru/crm/about). For a store with e-commerce, check how exactly the site sends the `purchase` event: its name alone does not prove that the money was received. [Sending e-commerce data](https://yandex.ru/support/metrica/ru/ecommerce/data).

### 4. Test the goal on a real scenario

1. Open the site with the parameter `_ym_debug=2`, for example `https://example.com/?_ym_debug=2`. If the address already has parameters, add `&_ym_debug=2`.
2. Perform the target action and check the needed counter and event in the Metrica debugger. If exclusion of your own visits is enabled, use a private window. [Official goal check](https://yandex.ru/support/metrica/ru/general/check-goal).
3. Match the sent event with a real request or a confirmed payment. Then test the error scenario: an unsent form or a canceled payment must not produce the same goal. This is a practical check that the result the user chose is what gets measured.
4. Find the goal by name or ID in the **Conversions (Конверсии)** section of Metrica, set the check period and look at the goal reaches and goal visits. Data appears after processing. [The "Conversions" report](https://yandex.ru/support/metrica/ru/reports/conversion).

Save the name, ID, counter, time of the check and its result. A direct test visit verifies that the goal works; it does not have to show up as a Direct ad conversion.

### 5. Build the campaign

In Direct, open **Add (Добавить) → Campaign (Кампанию) → Campaign Wizard (Мастер кампаний) → Conversions and traffic (Конверсии и трафик)** and enter the prepared link. Fill in the ads, **Sitelinks (Быстрые ссылки)**, region and ad schedule. In **Audience (Аудитория)** choose the suitable mode: **Select optimal (Подобрать оптимальную)** or **Configure manually (Настроить вручную)**. [Creating a campaign](https://yandex.ru/support/direct/ru/campaign-master/site).

Check everything Direct suggested automatically against the prepared assets. The region must match the service territory, and the schedule must match the ability to take requests. Evaluate automatic expansion and recommendations against the business goal.

The Wizard selects ad placements automatically. If you need to choose placements or exclude sites, pick a different tool before launch. [Placement options](https://yandex.ru/support/direct/ru/campaign-master/site).

### 6. Choose goals, strategy and budget

In **Promotion goal (Цель продвижения)** choose **Maximum target actions (Максимум целевых действий)**. Add the counter, the chosen **Target actions (Целевые действия)** and their prices. Check the Maps goals added automatically, if an organization is connected. [Campaign goals](https://yandex.ru/support/direct/ru/campaign-master/site).

The goal price means how much you are ready to pay for a result. Revenue from an order is a different quantity. Do not put it in the acquisition price field. If the acceptable price is not yet defined, prepare a calculation from the actual cost of a result in the history and the economics of the sale, then get the missing value from the user. State the inputs and assumptions; the average order value alone does not determine acceptable ad spend.

| Price option in the Wizard | How the ads are paid |
|---|---|
| Average price per week | Payment per click; the algorithm aims for the set average cost of a result |
| Fixed price | Payment per conversion; if several chosen goals are reached, each one is paid |

Match the payment mode to the chosen events. If a request and a payment can happen in the same visit, explain in advance the consequences of choosing both goals. [Wizard payment modes](https://yandex.ru/support/direct/ru/campaign-master/site).

Set the agreed **Weekly budget (Недельный бюджет)**. For a limited period, define the check and stop date separately: the weekly amount does not set the total spend over the whole time. Record the currency, VAT treatment and top-up amount according to the account interface. Take the minimum and the limits of the selected mode from the current form, not from an arbitrary example.

If **Budget adaptation (Адаптация бюджета)** is being considered, state separately the acceptable increase and the upper spending limit. This setting can raise next week's budget, so it must not be turned on silently when the user has a fixed budget. [Budget adaptation](https://yandex.ru/support/direct/ru/strategies/budget-adaptation).

A target price that is too low can limit the number of results. For a first conclusion about how an automatic strategy works, Yandex recommends 7-14 days; tracking of technical errors starts immediately. [How "Maximum conversions" works](https://yandex.ru/support/direct/ru/strategies/average-cpa).

### 7. Show the final settings and submit the campaign

Before the final action, fill in the table with actual data. It must be understandable to a person without reading code.

| Setting | Was | Will be |
|---|---|---|
| Account, campaign | Real data or "new campaign" | Name, type, account |
| Landing page | Current address or "not set" | Full verified address |
| Ads | Current variants or "not created" | Ready texts and files; changed months, prices and dates listed separately |
| Audience | Current settings or "not set" | Territory, schedule, audience selection mode |
| Counter and goals | Current choice or "not set" | Counter number; name and ID of each chosen goal |
| Price and payment | Current values or "not set" | Price of each goal, currency, chosen payment mode |
| Budget | Current budget or "not set" | Weekly amount; adaptation; agreed spending limit |
| Period | Current dates or "not set" | Start, check date, stop condition |
| Verification | What has already been checked | Who verifies the launch, the goals and the first results |

**Create campaign (Создать кампанию)** sends it to moderation; **Save as draft (Сохранить как черновик)** saves the preparation. Ads may start showing after approval if there is money in the account. [Moderation and launch](https://yandex.ru/support/direct/ru/campaign-master/site). If a top-up is required, the user chooses the method that suits them and checks the amount in the account. [Payment methods](https://yandex.ru/support/direct/ru/payments/payment-methods).

After submitting, record the campaign ID and its actual status. If it is rejected, read the reason for the specific element, prepare the fix and show it as before/after. After saving, check the settings against the table.

### 8. Check the ads after launch

Open **Statistics (Статистика)** under the campaign name, or **Statistics (Статистика) → Report Wizard (Мастер отчётов)**. Set the actual dates of operation and the same valuable goals. [Direct Report Wizard](https://yandex.ru/support/direct/ru/statistics/new-reports-wizard).

Check spend, impressions, clicks and conversions separately for each chosen goal. Through the skill you can get a report by campaign ID:

```sh
uv run scripts/report.py --account client-login --campaign 123456 \
  --preset campaigns --period LAST_7_DAYS --goals 20002,20003 \
  --attribution AUTO --no-cache
```

In the example the login, campaign and goals are placeholders; replace them with actual values. Bring attribution and VAT in line with the settings you use in the interface. Formulas and comparison details are in [REPORTS.md](REPORTS.md).

Compare conversions with accepted requests or paid orders for a comparable period. Account for processing delay and the time from the ad click to the result. If there are no conversions, first check whether the event works, whether the right ID is selected and whether data has had time to accumulate; only then discuss changes to the ads.

Give the user a concrete outcome: the campaign is created or prepared; its ID and status; the chosen goals; the actual budget and payment mode; what was checked; what action is still required and what data the result will be judged on.
