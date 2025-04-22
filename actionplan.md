# Action Plan: Migrating Agent Tool Logic to Python Backend

## 1. Identify Target Logic

**Target:**  
The `toolLogic` for the agent in `src/app/agentConfigs/customerServiceRetail/returns.ts`:
- `lookupOrders`
- `retrievePolicy`
- `checkEligibilityAndPossiblyInitiateReturn`

These are currently implemented as JavaScript functions (the last one even calls the OpenAI API via a Next.js route). They are invoked from the `useHandleServerEvent.ts` hook when the OpenAI Realtime API issues a function_call event.

## 2. Sequential Migration Steps

### Step 1: Set Up Python Backend

- Create a new directory, e.g., `python-backend/`.
- Initialize a Python project with a web framework (Flask or FastAPI recommended).
- Implement an endpoint `/execute-tool` that accepts POST requests with JSON: `{ "tool_name": ..., "arguments": ..., "call_id": ... }`.

### Step 2: Port Tool Logic to Python

- Implement Python equivalents for:
  - `lookupOrders` (returning mock order data)
  - `retrievePolicy` (returning the policy string)
  - `checkEligibilityAndPossiblyInitiateReturn` (replicating the logic, including calling OpenAI's API using the Python SDK)
- The endpoint should route requests to the correct function based on `tool_name` and return results in `{ "result": ... }`.

### Step 3: Create Next.js Proxy API Route

- In `src/app/api/`, create `execute-python-tool/route.ts`.
- This route should:
  - Accept POST requests from the frontend.
  - Forward the request to the Python backend.
  - Return the backend's response to the frontend.

### Step 4: Update AgentConfig

- In `src/app/agentConfigs/customerServiceRetail/returns.ts`:
  - Remove the `toolLogic` property (or at least the migrated entries).
  - Keep the `tools` array as-is.

### Step 5: Update Function Call Handling

- In `src/app/hooks/useHandleServerEvent.ts`:
  - In `handleFunctionCall`, add logic to detect when a tool call should be handled by the Python backend (i.e., if no local `toolLogic` exists and it's not `transferAgents`).
  - For these, POST to `/api/execute-python-tool` with the tool name, arguments, and call_id.
  - On response, send the result as a `function_call_output` event using `sendClientEvent`.

### Step 6: Test End-to-End

- Start both the Next.js app and the Python backend.
- Use the UI to trigger the relevant agent and tool calls.
- Confirm that:
  - The frontend sends the function call to the Next.js proxy.
  - The proxy relays it to the Python backend.
  - The backend executes the logic and returns the result.
  - The result is sent back to the OpenAI Realtime API and appears in the UI.

### Step 7: Document and Refine

- Update `system_overview.md` and the main README with:
  - Python backend setup instructions.
  - Any new environment variables or dependencies.
  - A note about the new architecture for tool logic.

---

**This plan is based on direct inspection of:**
- `src/app/agentConfigs/customerServiceRetail/returns.ts`
- `src/app/hooks/useHandleServerEvent.ts`
- `src/app/types.ts`
- `src/app/agentConfigs/utils.ts`
- The overall project structure

and is consistent with the high-level documentation in `system_overview.md`.