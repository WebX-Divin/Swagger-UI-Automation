# Swagger API Automation with Playwright

This repository contains Python utility functions for automating interactions with Swagger-based API documentation using Playwright. The provided functions allow you to execute API requests directly from the Swagger UI and, optionally, handle authorization tokens.

## Prerequisites

Before using this code, ensure you have the following installed:

- Python 3.7+
- Playwright library

To install Playwright and its dependencies, run:

```bash
pip install playwright
playwright install
```

## Code Overview

### 1. `interact_endpoint`

This function automates interactions with a specific Swagger API endpoint, enabling you to:

- Expand the endpoint section.
- Enable the "Try it out" feature.
- Provide a JSON request body (if applicable).
- Execute the request.
- Capture the response and extract tokens (optional).

```python
import json

async def interact_endpoint(page, endpoint_id, request_body=None, expect_token=False):

    token = None

    # Capture the response from the network request triggered by the "Execute" button
    async def capture_response(response):
        nonlocal token
        if response.status == 200 and expect_token:
            try:
                response_data = await response.json()
                # Extract token if it exists in the response
                token = response_data.get('data', {}).get('token', '')
            except Exception as e:
                print(f"Error extracting token: {e}")

    # Attach the listener for network responses
    page.on('response', capture_response)

    # Locate the section for the specified endpoint using its ID
    section_locator = page.locator(f"#{endpoint_id}")

    # Expand the section if not already expanded
    expanded = await section_locator.locator("button.opblock-control-arrow").get_attribute("aria-expanded")
    if expanded == "false":
        await section_locator.locator("button.opblock-control-arrow").click()

    # Click the "Try it out" button
    await section_locator.locator("button:has-text('Try it out')").click()

    # If a request body is provided, fill it in the text area
    if request_body is not None:
        request_body_locator = section_locator.locator("textarea.body-param__text")
        await request_body_locator.fill("")  # Clear existing content
        await request_body_locator.fill(json.dumps(request_body))
        await page.wait_for_timeout(2000)

    # Click the "Execute" button
    await section_locator.locator("button:has-text('Execute')").click()

    # Wait for the response
    await page.wait_for_timeout(5000)

    # Return the captured token, if any
    return token
```

#### Function Signature

```python
async def interact_endpoint(page, endpoint_id, request_body=None, expect_token=False):
```

#### Parameters

- `page`: The Playwright `page` object representing the browser page.
- `endpoint_id` (str): The HTML ID of the API endpoint section to interact with.
- `request_body` (dict, optional): JSON request body to send with the API request. Defaults to `None`.
- `expect_token` (bool, optional): Indicates if a token is expected in the API response. Defaults to `False`.

#### Usage

```python
await interact_endpoint(page, "operations-user-createUser", request_body={"name": "Divin Dass"}, expect_token=True)
```

## Detailed Steps

### `interact_endpoint`

1. Attach a network response listener to capture the API response.
2. Locate the API endpoint section using its HTML ID.
3. Expand the section if it is not already expanded.
4. Click the "Try it out" button to enable interaction.
5. If a request body is provided:
   - Clear the existing content in the request body field.
   - Fill the field with the provided JSON data.
6. Click the "Execute" button to send the API request.
7. Wait for the response and capture any token (if `expect_token=True`).
8. Return the captured token.

### 2. `authorize_with_token`

This function handles Swagger UI authorization by automating the process of pasting a bearer token into the authorization field.

``` python

async def authorize_with_token(page, token):
    """
    Authorizes Swagger UI by pasting the token into the "Authorize" input field.
    """
    if token:
        # Locate and click the "Authorize" button
        authorize_button = page.locator("button.btn.authorize.unlocked")
        await authorize_button.click()

        # Wait for the input field to appear
        await page.wait_for_selector("input#auth-bearer-value")

        # Fill the token in the input field
        auth_input = page.locator("input#auth-bearer-value")
        await auth_input.fill(token)

        # Click the "Authorize" button to confirm
        submit_button = page.locator("button[type='submit']")
        await submit_button.click()

        # Wait for the dialog to close
        close_button = page.locator("svg[viewBox='0 0 20 20']")
        await close_button.click()

        print("Authorization successful.")
    else:
        print("No token available for authorization.")
```

#### Function Signature

```python
async def authorize_with_token(page, token):
```

#### Parameters

- `page`: The Playwright `page` object representing the browser page.
- `token` (str): The bearer token for authorization.

#### Usage

```python
await authorize_with_token(page, token)
```

## Detailed Steps

### `authorize_with_token`

1. Locate and click the "Authorize" button in Swagger UI.
2. Wait for the authorization input field to appear.
3. Paste the token into the input field.
4. Submit the authorization form.
5. Close the authorization dialog.
6. Print a success message if authorization is completed.

### Run the Script

1. Ensure you have the required dependencies installed as described in the **Prerequisites** section.
2. Save the code examples into Python files (e.g., `swagger_playwright_utils.py` and `main.py`).
3. Run the main script:

   ```bash
   python main.py
   ```
