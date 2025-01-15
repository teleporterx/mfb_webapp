# About MFB Platform
> A Mutual Fund Broker Platform is an online application that allows users to invest in and manage mutual funds. Mutual funds are investment vehicles that pool money from many investors to purchase securities such as stocks, bonds, and other assets. A broker facilitates these investments by providing a platform where users can buy, sell, and track their mutual fund holdings.

The platform includes the following key features:

1. **Account Management:**
    
    Users can create accounts using email and password.
    They can log in securely to access their investment portfolio.

2. **Mutual Fund Selection:**

    Users can browse various mutual funds available in the market.
    The platform may allow users to search for **funds by family**, **scheme type** (open-ended or close-ended), asset class, risk level, and other criteria.

3. **Integration with APIs:**

    To fetch **real-time information**, fund data, and NAVs (Net Asset Values), the platform integrates with external APIs such as the **RapidAPI** marketplace to retrieve data about mutual funds.
    The RapidAPI integration provides a streamlined way to fetch updated mutual fund NAVs, allowing users to make informed decisions about their investments.

4. **Portfolio Management:**

    Users can **view their investment portfolio**, which includes details about all the mutual funds they’ve invested in.
    The platform provides **current value updates** and tracks the performance of the investments on an **hourly** basis.

5. **Security:**

    Strong authentication and authorization mechanisms are essential, including JWT (JSON Web Tokens) for secure login and token-based authorization.

---

## Fund Family House

> A fund family (or family of funds) refers to a collection of mutual funds or investment products managed by a single investment management company.

**Single Management Company:** A fund family includes all the separate funds managed by a particular investment firm, such as Vanguard, Fidelity, or BlackRock. Each fund within the family may have different investment objectives, asset classes, and risk profiles but is overseen by the same management team.

**Types of Funds:** Fund families typically offer a diverse range of investment options, including:
- Equity Funds: Focused on stocks.
- Fixed Income Funds: Concentrated on bonds.
- Balanced Funds: Combining stocks and bonds.
- Specialty Funds: Targeting specific sectors or asset classes.