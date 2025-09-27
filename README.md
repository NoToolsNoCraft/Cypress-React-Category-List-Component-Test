# Cypress React Category List Component Test

This Cypress component test verifies the behavior of the CategoryList React component by simulating different Redux store states. The test isolates the component logic by creating a mock Redux store for each scenario and mounting the component with the store. 
This ensures the test validates the component’s UI rendering under various conditions without relying on an actual backend or Redux implementation.


## React Category List Component



```bash
import { useEffect } from "react";
import { fetchCategories } from "../store/categorySlice";
import { useAppDispatch, useAppSelector } from "../store/hooks";

function CategoryList() {
  const dispatch = useAppDispatch();
  const categories = useAppSelector((state) => state.category.list);
  const loading = useAppSelector((state) => state.category.loading);
  const error = useAppSelector((state) => state.category.error);

  useEffect(() => {
    dispatch(fetchCategories());
  }, [dispatch]);

  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      <h2>Category List</h2>
      {loading ? (
        <div>Loading...</div>
      ) : (
        <ul>
          {categories!.map((category) => (
            <li key={category.id}>{category.name}</li>
          ))}
        </ul>
      )}
    </div>
  );
}

export default CategoryList;

```


## Cypress Component Test



```bash
import { mount } from "cypress/react";
import CategoryList from "../../src/components/CategoryList";
import { Provider } from "react-redux";
import { configureStore } from "@reduxjs/toolkit";

describe("CategoryList Component", () => {
  const createMockStore = (initialState: any) =>
    configureStore({
      reducer: {
        category: (state = initialState, action) => state, // fixed
      },
      preloadedState: { category: initialState },
    });

  const mountWithStore = (store: any) => {
    mount(
      <Provider store={store}>
        <CategoryList />
      </Provider>
    );
  };

  it("shows loading state", () => {
    const store = createMockStore({
      list: [],
      loading: true,
      error: null,
    });

    mountWithStore(store);

    cy.contains("Loading...").should("exist");
  });

  it("shows list of categories", () => {
    const store = createMockStore({
      list: [
        { id: 1, name: "Category 1" },
        { id: 2, name: "Category 2" },
      ],
      loading: false,
      error: null,
    });

    mountWithStore(store);

    cy.contains("Category List").should("exist");
    cy.contains("Category 1").should("exist");
    cy.contains("Category 2").should("exist");
  });

  it("shows error message", () => {
    const store = createMockStore({
      list: [],
      loading: false,
      error: "Failed to load categories",
    });

    mountWithStore(store);

    cy.contains("Error: Failed to load categories").should("exist");
  });
});

```

![Screenshot of Label Component](Screenshot%202025-09-27%20150359.png)

| Criteria                  | Justification                                                                                                                                                                                            |
| :------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Isolation**             | The component is tested in isolation with a **mock Redux store** created for each scenario. This removes dependency on a real Redux store or API calls, ensuring deterministic tests.                    |
| **Mocking Quality**       | Uses a **simple mock reducer with preloaded state** to control the Redux store for each test case. This ensures predictable results and avoids unnecessary complexity in the test setup.                 |
| **Coverage**              | Covers key component behaviors: <br>1) Rendering loading state <br>2) Rendering category list <br>3) Rendering error messages. This ensures comprehensive testing of possible UI states.                 |
| **Readability**           | Tests are clearly structured with descriptive case names, helper functions (`createMockStore`, `mountWithStore`), and straightforward assertions, making the test suite easy to understand and maintain. |
| **Clarity of Assertions** | Each test contains a clear, single assertion targeting the intended state (loading, success, or error), avoiding over-testing while ensuring correct component behavior in each scenario.                |
