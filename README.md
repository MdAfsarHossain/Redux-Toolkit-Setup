# [REDUX TOOLKIT](https://react-redux.js.org)

## [Redux Install](https://react-redux.js.org/tutorials/quick-start)

```npm
npm install @reduxjs/toolkit react-redux
```

## 📁 Folder Structure

```
redux-toolkit/
├── src/
│ ├── redux/
│ │ ├── api/
│ │ │ │ ├── baseApi.ts
│ │ ├── features/
│ │ │ │ ├── tasks/
│ │ │ │ │ ├── taskSlice.ts
│ │ ├── hooks.ts
│ │ └── store.ts
├── package.json
├── tsconfig.json
└── README.md
```

create store file `src/redux/store.ts`

```ts
import { configureStore } from "@reduxjs/toolkit";
import taskSlice from "./features/task/taskSlice";
import userSlice from "./features/user/userSlice";

const store = configureStore({
  reducer: {
    todos: taskSlice,
    users: userSlice,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

export default store;
```

add proverder in `src/main.ts` file

```ts
import { Provider } from "react-redux";
import store from "./redux/store.ts";

<Provider store={store}>
  <App />
</Provider>;
```

## [Typed Hooks]()

create type hooks file `src/redux/hooks.ts`

```ts
import { useDispatch, useSelector } from "react-redux";
import type { AppDispatch, RootState } from "./store";

export const useAppSelector = useSelector.withTypes<RootState>();
export const useAppDispatch = useDispatch.withTypes<AppDispatch>();
```

make slice file `src/redux/features/task/taskSlice.ts`

```ts
import type { RootState } from "@/redux/store";
import type { ITask } from "@/types";
import { createSlice, nanoid, type PayloadAction } from "@reduxjs/toolkit";
import { removeUser } from "../user/userSlice";
```

```ts
interface InitialState {
  tasks: ITask[];
  filter: "all" | "high" | "low" | "medium";
}

const initialState: InitialState = {
  tasks: [
    {
      id: "1",
      title: "Initialize Frontend",
      description: "Create Home Page, and routing",
      dueDate: "27-June-2025",
      isCompleted: false,
      priority: "high",
      assignedTo: null,
    },
    {
      id: "2",
      title: "Git Hub",
      description: "GitHub Reposotiroy study",
      dueDate: "27-June-2025",
      isCompleted: true,
      priority: "medium",
      assignedTo: null,
    },
  ],
  filter: "all",
};

type DraftTask = Pick<
  ITask,
  "title" | "description" | "dueDate" | "priority" | "assignedTo"
>;

const createTask = (taskData: DraftTask): ITask => {
  return {
    id: nanoid(),
    isCompleted: false,
    ...taskData,
    assignedTo: taskData.assignedTo ? taskData.assignedTo : null,
  };
};
```

```ts
const taskSlice = createSlice({
  name: "todo",
  initialState,
  reducers: {
    addTask: (state, action: PayloadAction<ITask>) => {
      // 1 Way to add data
      const id = uuidv4();
      const taskData = {
        ...action.payload,
        id,
        isCompletd: false,
      };
      state.tasks.push(action.payload);

      // 2nd Way to add data
      const taskData = createTask(action.payload);
      state.tasks.push(taskData);
    },

    toggleCompleteState: (state, action: PayloadAction<string>) => {
      state.tasks.forEach((task) =>
        task.id === action.payload
          ? (task.isCompleted = !task.isCompleted)
          : task
      );
    },

    deleteTask: (state, action: PayloadAction<string>) => {
      state.tasks = state.tasks.filter((task) => task.id !== action.payload);
    },

    updateFilter: (
      state,
      action: PayloadAction<"low" | "medium" | "high" | "all">
    ) => {
      state.filter = action.payload;
    },
  },

  extraReducers: (builder) => {
    builder.addCase(removeUser, (state, action) => {
      state.tasks.forEach((task) =>
        task.assignedTo === action.payload ? (task.assignedTo = null) : task
      );
    });
  },
});

export const { addTask, toggleCompleteState, deleteTask, updateFilter } =
  taskSlice.actions;

export default taskSlice.reducer;
```

### [Selector Functions]()

`src/redux/features/task/taskSlice.ts`

```ts
// Selector Functions
export const selectFilter = (state: RootState) => {
  return state.todos.filter;
};
```

### [Get All Tasks]()

`src/pages/Task.tsx`

```ts
import { useAppDispatch, useAppSelector } from "@/redux/hooks";

// Selector Functions
const tasks = useAppSelector(selectTasks);
const dispatch = useAppDispatch();
```

## [React Redux TypeScript Quick Start](https://react-redux.js.org/tutorials/typescript-quick-start)

## [RTK Query]()

`src/redux/api/baseApi.ts`

```ts
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query/react";

export const baseApi = createApi({
  reducerPath: "baseApi",
  baseQuery: fetchBaseQuery({ baseUrl: "http://localhost:5000/api" }),
  tagTypes: ["task"],
  endpoints: (builder) => ({
    getTasks: builder.query({
      query: () => "/tasks",
      providesTags: ["task"],
    }),
    createTask: builder.mutation({
      query: (taskData) => ({
        url: "/tasks",
        method: "POST",
        body: taskData,
      }),
      invalidatesTags: ["task"],
    }),
    deleteTask: builder.mutation({
      query: (taskId) => ({
        url: `/tasks/${taskId}`,
        method: "DELETE",
        body: taskId,
      }),
    }),
  }),
});

export const {
  useGetTasksQuery,
  useCreateTaskMutation,
  useDeleteTaskMutation,
} = baseApi;
```

create store file `src/redux/store.ts`

```ts
import { configureStore } from "@reduxjs/toolkit";
import { baseApi } from "./api/baseApi";
import taskSlice from "./features/task/taskSlice";
import userSlice from "./features/user/userSlice";

const store = configureStore({
  reducer: {
    todos: taskSlice,
    users: userSlice,
    [baseApi.reducerPath]: baseApi.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(baseApi.middleware),
});

export default store;
```

`src/redux/features/Tasks.tsx`

```ts
const { data, isLoading } = useGetTasksQuery(undefined, {
  pollingInterval: 3000,
  refetchOnFocus: true,
  refetchOnMountOrArgChange: true,
  refetchOnReconnect: true,
});
```

## [Pedro Tech](https://www.youtube.com/watch?v=sKJU5_3Vx0A)

`src/store.ts`

```ts
import { configureStore } from "@reduxjs/toolkit";
import { setupListeners } from "@reduxjs/toolkit/query";
import { jsonPlaceholderApi } from "./services/jsonPlaceholderApi";

export const store = configureStore({
  reducer: {
    [jsonPlaceholderApi.reducerPath]: jsonPlaceholderApi.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(jsonPlaceholderApi.middleware),
});

setupListeners(store.dispatch);
```

`src/main.tsx`

```ts
import React from "react";
import ReactDOM from "react-dom/client";
import { Provider } from "react-redux";
import { ApiProvider } from "@reduxjs/toolkit/query/react";
import { store } from "./store";
import App from "./App";
import "./index.css";

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
);
```

`src/services/jsonPlaceHolderApi.ts`

```ts
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query/react";

export const jsonPlaceholderApi = createApi({
  reducerPath: "jsonPlaceholderApi",
  baseQuery: fetchBaseQuery({
    baseUrl: "https://jsonplaceholder.typicode.com/",
  }),
  refetchOnFocus: true,
  endpoints: (builder) => ({
    // Query to fetch posts
    getPosts: builder.query({ query: () => "posts" }),
    createPosts: builder.mutation({
      query: (newPost) => ({
        url: "posts",
        method: "POST",
        body: newPost,
      }),
    }),
  }),
});

export const { useGetPostsQuery, useCreatePostsMutation } = jsonPlaceholderApi;
```

`src/App.tsx`

```ts
import { useState } from "react";
import "./App.css";
import {
  useGetPostsQuery,
  useCreatePostsMutation,
} from "./services/jsonPlaceholderApi";

function App() {
  const [newPost, setNewPost] = useState({ title: "", body: "", id: 99999 });

  const { data, error, isLoading, refetch } = useGetPostsQuery();
  const [createPost, { isLoading: isCreating, error: createError }] =
    useCreatePostsMutation();

  if (isLoading) return <p> Loading...</p>;

  if (createError) return <p> There was an error creating a post</p>;

  if (error) return <p> There was an error :(</p>;

  const handleCreatePost = async () => {
    await createPost(newPost);
    refetch();
  };

  return (
    <>
      <h1> Leave A Like :) </h1>

      <div>
        <input
          type="text"
          placeholder="Title..."
          onChange={(e) =>
            setNewPost((prev) => ({ ...prev, title: e.target.value }))
          }
        />
        <input
          type="text"
          placeholder="Body..."
          onChange={(e) =>
            setNewPost((prev) => ({ ...prev, body: e.target.value }))
          }
        />
        <button onClick={handleCreatePost} disabled={isCreating}>
          {" "}
          Create Post{" "}
        </button>
      </div>

      <div>
        {data?.map((post) => (
          <p> {post.title}</p>
        ))}
      </div>
    </>
  );
}

export default App;
```
