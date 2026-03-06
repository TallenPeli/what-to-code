# API Documentation

**Base URL:** `http://localhost:8080`

**Content-Type:** `application/json`

## Global Response Shape

Every response from this API follows a consistent wrapper structure. This makes it easy to handle errors in your React `fetch` or `axios` logic.

```typescript
{
  "status": boolean,        // true if successful, false otherwise
  "data": any | null,       // the payload (array of Ideas or single Idea)
  "error": string | null    // error message if status is false
}

```

---

## 1. Create a New Idea

Adds a new idea to the database.

- **Endpoint:** `POST /post-idea`
- **Rate Limit:** 6 requests per second (burst capacity of 24). If exceeded, returns `429 Too Many Requests`.
- **Max Body Size:** 1MB.

### Request Body

The `id` is server-generated, so you should **not** send it from the frontend.

| Field   | Type     | Description                                  |
| ------- | -------- | -------------------------------------------- |
| title   | string   | The title of the idea.                       |
| content | string   | Detailed description.                        |
| tags    | string[] | An array of strings (e.g., `["web", "go"]`). |

**Example:**

```json
{
  "title": "Build a React App",
  "content": "Create a frontend for my Go API.",
  "tags": ["react", "frontend", "learning"]
}
```

### Response

- **Success (200 OK):** `{"status": true}`
- **Rate Limited (429):** `{"status": false, "error": "rate limit exceeded..."}`
- **Error (500):** `{"status": false, "error": "internal server error"}`

---

## 2. Get Ideas (Search / List)

Retrieves ideas based on ID, tags, or a general random list.

- **Endpoint:** `GET /ideas`
- **Query Parameters:**
- `id` (int): If provided, returns a specific idea. **Overwrites other parameters.**
- `tags` (string): A comma-separated list (e.g., `react,go`).
- `limit` (int): The number of results to return. Defaults to `1`.

### Scenario A: Fetch by ID

If you pass an `id`, the API returns an array containing that single idea.

**Example:** `GET /ideas?id=5`

**Success Response:**

```json
{
  "status": true,
  "data": [
    {
      "id": 5,
      "title": "Build a React App",
      "content": "...",
      "tags": ["react", "learning"]
    }
  ]
}
```

### Scenario B: Fetch by Tags

If you pass `tags`, the API returns ideas that contain **all** of the specified tags, ordered randomly.

**Example:** `GET /ideas?tags=react,frontend&limit=10`

### Scenario C: General Random List

If no `id` or `tags` are provided, the API returns a random selection of ideas up to the `limit`.

**Example:** `GET /ideas?limit=5`

---

## Data Models (for your Typescript types)

If you are using TypeScript in your React project, you can use these definitions:

```typescript
interface Idea {
  id: number;
  title: string;
  content: string;
  tags: string[];
}

interface ApiResponse<T> {
  status: boolean;
  data?: T;
  error?: string;
}
```

## Quick Integration Tips:

1. **URLSearchParams:** Use the `URLSearchParams` web API to build your `GET /ideas` queries safely, especially when handling multiple tags.
2. **Handling the Data Shape:** Note that `GET /ideas` **always** returns an array in the `data` field, even when you are fetching a single idea by `id`. In your React state, you'll likely want to do `result.data[0]` when fetching by ID.
3. **Error Handling:** Check `response.ok` first, but then check `data.status` to see if the backend logic failed (like a DB error).
