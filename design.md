# Overall Design

**Use GitHub Actions secrets for all sensitive values.**

## Unraid GraphQL API

- Play with the developer sandbox and determine which Unraid data I want to share publicly
    - Observe returned responses, test and experiment

# Status Ideas
- Number of containers currently running
    - This avoids container names and showing status for reach of them
- Storage array used vs total, either as a percentage or in TB
- Hardware information (i.e. cpu cores, memory, types of devices, etc.)
- UPS information (i.e. model, load, etc.)
- Number of VMs
- Server uptime

## Python Retrieval

- Use the `requests` library; consider Pydantic after first understanding and implementing the validation manually
- Retrieve only the selected data from the GraphQL endpoint
- Validate the GraphQL response and required values
- Transform the selected fields into a small public output structure
- Include a collection timestamp so the site can show the age of the snapshot
- Write the transformed data to a temporary `status.json`

## GitHub Actions

### WireGuard Setup

- Create a dedicated WireGuard peer for GitHub Actions
- Restrict the peer to the required Unraid address and GraphQL port
- Store the WireGuard private key or configuration in GitHub Actions secrets

### Run Python Script

- Python script retrieves structured JSON from GraphQL API endpoint
- Creates a temporary `status.json` in the GitHub Actions runner workspace
- Stops without uploading if retrieval, validation, or transformation fails

### Upload Status to Cloudflare R2

- Upload `status.json` as an R2 Standard object only after successful validation
- Set the content type to `application/json` and configure appropriate cache behavior
- Preserve the last valid R2 object when a workflow run fails

### Astro Site — Separate Repository

- Use a small browser script to retrieve `status.json` from R2 when the page loads
- Update existing HTML elements with the returned values
- Display the collection timestamp and a clear loading or failure state
- Configure R2 CORS to allow the portfolio site's domain

## Testing

- Start with a fabricated GraphQL response rather than a real Unraid retrieval
- Test transformation, missing fields, invalid values, and malformed responses
- Confirm failures do not overwrite the last valid R2 object
- Confirm the portfolio site can retrieve the R2 object through its CORS policy
- Test CDN caching to confirm that a browser refresh receives an acceptably recent snapshot
