# Grand finale: Patientor

## Working with an existing codebase

When diving into an existing codebase for the first time, it is good to get an overall view of the conventions and structure of the project. You can start your research by reading the *README.md* in the root of the repository. Usually, the README contains a brief description of the application and the requirements for using it, as well as how to start it for development. If the README is not available or someone has "saved time" and left it as a stub, you can take a peek at the *package.json*. It is always a good idea to start the application and click around to verify you have a functional development environment.

You can also browse the folder structure to get some insight into the application's functionality and/or the architecture used. These are not always clear, and the developers might have chosen a way to organize code that is not familiar to you. The sample project used in the rest of this part is organized, feature-wise. You can see what pages the application has, and some general components, e.g. modals and state. Keep in mind that the features may have different scopes. For example, modals are visible UI-level components whereas the state is comparable to business logic and keeps the data organized under the hood for the rest of the app to use.

TypeScript provides types for what kind of data structures, functions, components, and state to expect. You can try looking for *types.ts* or something similar to get started. VSCode is a big help and simply highlighting variables and parameters can provide quite a lot of insight. All this naturally depends on how types are used in the project.

If the project has unit, integration, or end-to-end tests, reading those is most likely beneficial. Test cases are your most important tool when refactoring or adding new features to the application. You want to make sure not to break any existing features when hammering around the code. TypeScript can also give you guidance with argument and return types when changing the code.

Remember that reading code is a skill in itself, so don't worry if you don't understand the code on your first readthrough. The code may have a lot of corner cases, and pieces of logic may have been added here and there throughout its development cycle. It is hard to imagine what kind of problems the previous developer has wrestled with. Think of it like growth rings in trees. Understanding everything requires digging deep into the code and business domain requirements. The more code you read, the better you will be at understanding it. You will most likely read far more code than you are going to produce throughout your life.

## Patientor frontend

Before diving into the code, let us start both the frontend and the backend.

If all goes well, you should see a patient listing page. It fetches a list of patients from our backend, and renders it to the screen as a simple table. There is also a button for creating new patients on the backend. As we are using mock data instead of a database, the data will not persist - closing the backend will delete all the data we have added. UI design has not been a strong point of the creators, so let's disregard the UI for now.

After verifying that everything works, we can start studying the code. All the interesting stuff resides in the *src* folder. For your convenience, there is already a *types.ts* file for basic types used in the app, which you will have to extend or refactor in the exercises.

In principle, we could use the same types for both backend and frontend, but usually, the frontend has different data structures and use cases for the data, which causes the types to be different. For example, the frontend has a state and may want to keep data in objects or maps whereas the backend uses an array. The frontend might also not need all the fields of a data object saved in the backend, and it may need to add some new fields to use for rendering.

In the folder structure, besides the component `App` and a directory for services, there are currently three main components: `AddPatientModal` and `PatientListPage` which are both defined in a directory, and a component `HealthRatingBar` defined in a file. If a component has some subcomponents not used elsewhere in the app, it might be a good idea to define the component and its subcomponents in a directory. For example, now the AddPatientModal is defined in the file `components/AddPatientModal/index.tsx` and its subcomponent `AddPatientForm` in its own file under the same directory.

There is nothing very surprising in the code. The state and communication with the backend are implemented with `useState` hook and Axios, similar to the notes app in the previous section. Material UI is used to style the app and the navigation structure is implemented with React Router.

From a typing point of view, there are a couple of interesting things. Component `App` passes the function `setPatients` as a prop to the component `PatientListPage`:
```
const App = () => {
  const [patients, setPatients] = useState<Patient[]>([]);s
  // ...
  
  return (
    <div className="App">
      <Router>
        <Container>
          <Routes>
            // ...
            <Route path="/" element={
              <PatientListPage
                patients={patients}
                setPatients={setPatients}
              />}
            />
          </Routes>
        </Container>
      </Router>
    </div>
  );
};
```

To keep the TypeScript compiler happy, the props should be typed as follows:
```
interface Props {
  patients : Patient[]
  setPatients: React.Dispatch<React.SetStateAction<Patient[]>>
}

const PatientListPage = ({ patients, setPatients } : Props ) => { 
  // ...
}
```

So the function `setPatients` has type `React.Dispatch<React.SetStateAction<Patient[]>>`. We can see the type in the editor when we hover over the function.

The React TypeScript cheatsheet has a pretty nice list of typical prop types, where we can seek for help if finding the proper typing for props is not obvious.

`PatientListPage` passes four props to the component `AddPatientModal`. Two of these props are functions. Let us have a look how these are typed:
```
const PatientListPage = ({ patients, setPatients } : Props ) => {

  const [modalOpen, setModalOpen] = useState<boolean>(false);
  const [error, setError] = useState<string>();

  // ...

  const closeModal = (): void => {
    setModalOpen(false);
    setError(undefined);
  };

  const submitNewPatient = async (values: PatientFormValues) => {
    // ...
  };
  // ...

  return (
    <div className="App">
      // ...
      <AddPatientModal
        modalOpen={modalOpen}
        onSubmit={submitNewPatient}
        error={error}
        onClose={closeModal}
      />
    </div>
  );
};
```

Types look like the following:
```
interface Props {
  modalOpen: boolean;
  onClose: () => void;
  onSubmit: (values: PatientFormValues) => Promise<void>;
  error?: string;
}

const AddPatientModal = ({ modalOpen, onClose, onSubmit, error }: Props) => {
  // ...
}
```

`onClose` is just a function that takes no parameters, and does not return anything, so the type is `() => void`.

The type of `onSubmit` is a bit more interesting, it has one parameter that has the type `PatientFormValues`. The return value of the function is `Promise<void>`. So again, the function type is written with the arrow syntax:
```
(values: PatientFormValues) => Promise<void>
```

The return value of an `async` function is a promise with the value that the function returns. Our function does not return anything so the proper return type is just `Promise<void>`.

## Full entries

