# Brief

This `mdbook` has been hosted to provide the documentation, setup and usage for the **Mutual Fund Broker Web Application with RapidAPI Integration** in a simplistic and programmatic manner.

> If you want to dive right in and and test the application locally, refer [Setup & Usage](Setup_&_Usage.md) and visit `http://localhost:5000/login` after setup.

- The **Frontend** and **Docker** sections are **greyed out** as they don't have a root page like [Backend](2_Backend/2_1_Feature_Map.md). The sub-sections are still accessible via the sidebar dropdown (`>`) option.

---

## Content Breakdown

- [About MFB Platform](About_MFB_Platform.md) gives an outline and understanding of what the application is all about.

- The [Check List](Check_List.md) has been updated with the completion status.

- Links to the Github repositories containing the source code and the Postman collection used for testing the backend API has been provided in [Important Links](Important_Links.md). The repos have not been updated with a detailed readme as most of the documentation is present here.

- The setup & configuration of the application is detailed in [Setup & Usage](Setup_&_Usage.md).

- There are **no dummy credentials** hardcoded into the system. In fact the application has a **fully functional registration and login system**. So for testing, one can **register with an email and password**.

- I have explained about why we needn't have **database migration scripts or setup** in [Database Setup](2_Backend/2_2_Database_Setup.md)
    - In simple terms I don't need to **enforce a schema** because I'm using **MongoDB**

- Some elements of **SDLC** and **DevSecOps** have been followed/used
    - Brainstorming on schema and endpoint route structure using **Miro** & **Draw.io**
    - Meaningful Github commits (for feature addition) [Ex: Commits for mfb_webapp_backend](https://github.com/teleporterx/mfb_webapp_backend/commits/main/)
    - Didn't use branches and PRs because of solo development
    - Secure coding practices with `.config` for loading secrets from environment
    - Orchestration with `docker-compose`

- **E2E tests** have been included for the endpoints in the documentation followed by their implementation.

- Reuse of code has been done wherever possible using modular design such as segregating reusable snips into services and models.

- Implemented the **buy functionality**, **portfolio** and **hourly NAV tracking update**.

**Backend**

- The **RapidAPI Integration** is **fully functional**, however there are some workarounds in the code for **API savings** (using _cached data_, to not exhaust the free tier API calls).'

- **API versioning** and **sub-router** techniques have been used to make the API more **scalable** and **maintainable**.

- The endpoints beyond the **login** and **register** are **protected** by **JWT authentication**.

- The RapidAPI key needs to be specified differently for each of the deployment methods
    - If you are using `docker-compose`, you'd need to include it in the `ENV` specifications of the backend container of the compose file.
    - For manual setup, you'd need to add it to the `.env` file in the backend project root
    
    (Refer [Setup & Usage](Setup_&_Usage.md) for specifics)

**Frontend**

- The `/` (root) page has been left as the default as the frontend would only be evaluated for understanding of the client server model.
- So you can start local testng from `/register` and `/login`.
- The frontend has been built using **NextJs** and **React** and is **fully functional** .
- All pages provide a **smooth user experience** with **validation** and **error handling** (both native and from the backend API)
- Session handling is done using **local storage** of the acquired **JWT** during login for simplicity and **validation** via backend API calls (**check_login endpoint**).
- The the dashboard page when loaded fetches the list of **mutual fund families** offering **open ended schemes** from the backend API and displays them in a scrollable dropdown.
- Users can then select a **scheme** and view its **details** (including **NAV**, **ISIN (Growth)** and **ISIN (Reinvestment)** as a pop-up. Hover animation as well as click out of pop-up to close has been implemented for a seamless user experience.
- The pop-up has a **counter field** and a **buy button** which would call the backend to **simulate a buy transaction**.
- The user can then view the **portfolio** which gives a list of schemes they have purchased some units of and other relevant details.