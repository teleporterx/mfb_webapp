# Rapid API Integration & Protected Endpoints

![Rapid Plan](assets/rapid_plan.png)

## Things to consider:

- How large is the response data?

    **50,000 items returned** per month hard limit would likely be an **issue** and the API would be **exhausted quickly** if the response json data is **large (which is the actual case).**

- How many requests would be made?

    The predicted request rate (for a demo use case) is **low to moderate** which doesn't pose as a major concern because of the 1000 requests per hour limit.

---

## 1. Exhaustive Implementation

**RapidAPIService to get all the open ended schemes**
```python
# /api/v1/services/rapidapi_mutfund.py

import httpx
from api.v1.config import CONFIG

class RapidAPIService:
    @staticmethod
    async def fetch_latest_open_ended_schemes():
        """
        Fetch latest schemes for the selected fund family from the /latest endpoint.
        Fetches all mutual fund schemes without any built-in filtering 
        """
        url = f"{CONFIG['RAPID_URL']}/latest?Scheme_Type=Open"  # Appending /latest to the base URL
        headers = {
            "X-RapidAPI-Key": CONFIG["RAPID_MUT_FUND_KEY"],
            "X-RapidAPI-Host": "latest-mutual-fund-nav.p.rapidapi.com"
        }
        # params = {"RTA_Agent_Code": fund_family}

        async with httpx.AsyncClient() as client:
            response = await client.get(url, headers=headers)
            response.raise_for_status()  # Raise an exception for HTTP errors
            return response.json()
```

**Filtering the response to provide the data only for the requested fund family**
```python
@router.get("/fund_schemes/latest/open_ended")
async def get_open_ended_latest_schemes(
    request: FundFamilyRequest,
    authorization: str = Header(None)
):
    """
    Fetch latest schemes for the selected fund family using the /latest endpoint.
    """
    if authorization is None:
        raise HTTPException(status_code=401, detail="Authorization token is missing.")

    # Extract the token from "Bearer <token>"
    token_prefix = "Bearer "
    if not authorization.startswith(token_prefix):
        raise HTTPException(status_code=401, detail="Invalid authorization header format.")
    
    token = authorization[len(token_prefix):]  # Get the actual token

    try:
        # Validate and decode the token
        current_user = AuthSecurity.get_current_user(token)

        # Fetch data from RapidAPI
        all_schemes = await RapidAPIService.fetch_latest_open_ended_schemes()

        # Filter open-ended schemes for the specific fund family
        ff_open_ended_schemes = [
            scheme for scheme in all_schemes
            if scheme.get("Mutual_Fund_Family") == request.fund_family
            # if scheme.get("Scheme_Type") == "Open Ended Schemes"
            # and scheme.get("Mutual_Fund_Family") == request.fund_family
        ]

        if not ff_open_ended_schemes:
            raise HTTPException(status_code=404, detail="No open-ended schemes found for the given fund family.")

        return {"status": "success", "data": ff_open_ended_schemes}
    except HTTPException as e:
        raise e  # Re-raise HTTP exceptions
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Internal server error: {str(e)}")
```

## 2. Addressing API Limits

The hard limit of 50000 items is reached faster than the 1000 requests per hour, so after just a handful of successful requests, you're running into the limit.

**Possible Solutions:**
- Check API documentation or usage to see if it implements native filtering

![Mutual Fam Query](assets/rapid_mutfam_filter.png)

And it does!...

- For real-time data this can be used.

- However, for listing out all the open ended schemes we'd need to go for the exhaustive implementation.

- Or, another possible solution for listing out the open ended schemes is to use caching.

## 3. Endpoints Implementation

### 3.1. Fund Families (GET /v1/fund_families)

**Purpose:** Fetch all open ended schemes and filter all families using the /latest endpoint.

### 3.2. Open Ended Schemes for a Fund Family (GET /v1/fund_schemes/latest/open_ended)

**Purpose:** Filter out selected fund family and associated details.

**Request**: `authorization Bearer <token>`