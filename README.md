_*When editing, right-click "README.md" file to select "Open preview" to view the translated format simultaneously.*_

# TABLE OF CONTENTS

1. [Front-end Build](#front-end)
2. [Back-end Build](#back-end)
3. [Typical Dependencies to Install](#dependencies)
4. [Package.json Script](#package.json)
5. [Back-End Architecture](#backarchitecture)
6. [Front-End Architecture](#frontarchitecture)
7. [Common Errors w/Troublshooting](#errors)
8. [CSS Rules / Tricks](#css)
9. [React](#react)

# Front-end Build <a name="front-end"></a>

COMMAND | DESCRIPTION

- npx create-react-app \<file-name\> | Creates a react app with boiler plate ((Perhaps a suggestion it be named "client"?))
- npm i axios | Promise based HTTP client for the browser and node.js
- npm i react-router-dom | DOM bindings for React Router

# Back-end Build <a name="back-end"></a>

COMMAND | DESCRIPTION

- npm init -y | Creates package.json
- npm i | Node modules
- npm i nodemon -D | Automatically restarts node application when files change
- touch **.gitignore** | Instructs Git specifically what to ignore
  - echo "**.DS_Store**" >> .gitignore
  - echo "**node_modules**" >> .gitignore

### Typical dependencies to install<a name="dependencies"></a>

- npm i body-parser | Node.js body parsing middleware
- npm i cors | CORS is a node.js package for providing a Connect/Express middleware that can be used to enable CORS with various options.
- npm i express | Web framework for node
- npm i mongoose | Elegant mongodb object modeling for node.js
- npm i morgan | HTTP request logger middleware for node.js

#### Package.json Script<a name="package.json"></a>

```
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node server.js",
    "dev": "nodemon server.js"
  },

```

### Back-End Architecture<a name="backarchitecture"></a>

FOLDER | FILE(s) | DESCRIPTION

- db | connection.js | Defining connection
<details>
 <summary>Expand Boilerplate</summary>

```
const mongoose = require("mongoose");

let MONGODB_URI =
  process.env.PROD_MONGODB || "mongodb://127.0.0.1:27017/app-nameDB"; // change this to whatever you actually want to name your database

mongoose
  .connect(MONGODB_URI, { useUnifiedTopology: true, useNewUrlParser: true })
  .then(() => console.log("Successfully connected to MongoDB."))
  .catch((e) => console.error("Connection error", e.message));

module.exports = mongoose.connection;

```

</details>

- controllers | \<variables\>.js | CRUD functions
<details>
 <summary>Expand Boilerplate</summary>

```
const Variable = require("../models/variable");
const db = require("../db/connection");

db.on("error", console.error.bind(console, "MongoDB connection error:"));

const getVariables = async (req, res) => {
  try {
    const variables = await Variable.find();
    res.json(variables);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

const getVariable = async (req, res) => {
  try {
    const { id } = req.params;
    const variable = await Variable.findById(id);
    if (variable) {
      return res.json(variable);
    }
    res.status(404).json({ message: "Variable not found!" });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

const createVariable = async (req, res) => {
  try {
    const variable = await new Variable(req.body);
    await variable.save();
    res.status(201).json(variable);
  } catch (error) {
    console.log(error);
    res.status(500).json({ error: error.message });
  }
};

const updateVariable = async (req, res) => {
  const { id } = req.params;
  await Variable.findByIdAndUpdate(id, req.body, { new: true }, (error, variable) => {
    if (error) {
      return res.status(500).json({ error: error.message });
    }
    if (!variable) {
      return res.status(404).json({ message: "Variable not found!" });
    }
    res.status(200).json(variable);
  });
};

const deleteVariable = async (req, res) => {
  try {
    const { id } = req.params;
    const deleted = await Variable.findByIdAndDelete(id);
    if (deleted) {
      return res.status(200).send("Variable deleted");
    }
    throw new Error("Variable not found");
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

module.exports = {
  createVariable,
  getVariables,
  getVariable,
  updateVariable,
  deleteVariable,
};

```

</details>

- models | \<variable\>.js | Schema
<details>
 <summary>Expand Boilerplate</summary>

```
const mongoose = require("mongoose");
const Schema = mongoose.Schema;

const Variable = new Schema(
  {
    title: { type: String, required: true },
    imgURL: { type: String, required: true },
    content: { type: String, required: true },
    author: { type: String, required: true },
  },
  { timestamps: true }
);

module.exports = mongoose.model("variables", Variable);

```

</details>

- routes | \<variables\>.js | API endpoints
<details>
 <summary>Expand Boilerplate</summary>

```
const { Router } = require("express");
const controllers = require("../controllers/<filename>");

const router = Router();

router.get("/<variables>", controllers.getVariables);
router.get("/<variables>/:id", controllers.getVariable);
router.post("/<variables>", controllers.createVariable);
router.put("/<variables>/:id", controllers.updateVariable);
router.delete("/<variables>/:id", controllers.deleteVariable);

module.exports = router;

```

</details>

- seed | \<variables\>.js | Data to populate initial database
<details>
 <summary>Expand Boilerplate</summary>

```
const db = require("../db/connection");
const Variable = require("../models/variable");

db.on("error", console.error.bind(console, "MongoDB connection error:"));

const main = async () => {
  const variables = [
    {
    }
    ];
  await Variable.insertMany(variables);
  console.log("Created vaiables!");
  };
const run = async () => {
await main();
db.close();
};

run();

```

</details>

- N/A | server.js | Combined variables to set connection to database

<details>
 <summary>Expand Boilerplate</summary>
   
   ```
const express = require("express");
const cors = require("cors");
const bodyParser = require("body-parser");
const logger = require("morgan");
const variableRoutes = require("./routes/variable");
const db = require("./db/connection");
const { response } = require("express");
const PORT = process.env.PORT || 3000;

const app = express();

app.use(cors());
app.use(bodyParser.json());
app.use(logger("dev"));

app.use("/api", variableRoutes);

db.on("error", console.error.bind(console, "MongoDB connection error:"));

app.listen(PORT, () => console.log(`Listening on port: ${PORT}`));

app.get("/", (req, res) => res.send("This is root!"));

```

</details>

### Front-End Architecture<a name="frontarchitecture"></a>

All of this is within the `client` folder // or whatever you named your react app folder

FOLDER | FILE(s) | DESCRIPTION
* public | favicon.ico, logo192.png, logo512.png, assets | Mostly images
Favicon is the image which appears on the left edge of the tab when your app is running. The logos appear elsewhere. `assets` is a folder for any media like images or sounds you'll use in your app (apart from anything you link to externally).

* src | Various | The core of your app
This will contain most of what your app displays.

<details>
 <summary>All of the below files and folders will be within `src`.</summary>

* NA | index.js | Where the computer will look first. The included boilerplate allows you to use routing.
<details>
 <summary>Expand Boilerplate</summary>

```

import React from "react";
import ReactDOM from "react-dom";
import "./index.css";
import App from "./App";
import reportWebVitals from "./reportWebVitals";
import { BrowserRouter as Router } from "react-router-dom";

ReactDOM.render(
<React.StrictMode>
<Router>
<App />
</Router>
</React.StrictMode>,
document.getElementById("root")
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();

```
</details>

* NA | App.js | Controls the traffic. Uses Switch which only sends users to the first matching Screen
<details>
 <summary>Expand Boilerplate</summary>

```

import React from "react";
import "./App.css";
import { Route, Switch } from "react-router-dom";
import <ScreenName> from "./screens/<ScreenName>/<ScreenName>";
import { getUser } from "./services/Users";

const App = () => {
return (

<div className="App">
<Switch>
<Route exact path="/" component={<ScreenName>} />
</Switch>
</div>
);
};

export default App;

```
</details>

* screens | ScreenName | Screens are components which take up essentially the entire body.
Each folder is named in PascalCase and contains two files. Each file has the exact same name as the folder. One uses the .css extension, the other uses the .jsx extension. Below is an example of the ScreenName.jsx file. Note: You can populate it rapidly by typing `rsf` and hitting enter, but remember you'll have to add the `import './ScreenName.css` manually.
<details>
 <summary>Expand Boilerplate</summary>

```

import React from "react";
import <ComponentName> from "../../components/<ComponentName>/<ComponentName>";

export default function ScreenName(props) {
return (

<div>
<ComponentName />
</div>
);
}

```
</details>

* components | ComponentName | Components represent one element on the screen.
Each folder is named in PascalCase and contains two files. Each file has the exact same name as the folder. One uses the .css extension, the other uses the .jsx extension. Below is an example of the ComponentName.jsx file. Note: You can populate it rapidly by typing `rsf` and hitting enter, but remember you'll have to add the `import './ComponentName.css` manually.
<details>
 <summary>Expand Boilerplate</summary>

```

import React from "react";
import <OtherComponent> from "../../<OtherComponent>/<OtherComponent>";

export default function ComponentName(props) {
return (

<div>
<OtherComponent />
</div>
);
}

```
</details>

* services | apiConfig.js | Exports a function to let the client connect to the database
<details>
 <summary>Expand Boilerplate</summary>

```

import axios from 'axios'

let apiUrl

const apiUrls = {
production: 'https://<App-Name>.herokuapp.com/api', // Make sure you update this with the actual name of your Heroku app
development: 'http://localhost:3000/api'
}

if (window.location.hostname === 'localhost') {
apiUrl = apiUrls.development
} else {
apiUrl = apiUrls.production
}

const api = axios.create({
baseURL: apiUrl
})

export default api

```
</details>

* services | <Collection>.js | Exports functions to use on the object types defined in your models.
 Note, by default it contains the CRUD functions, but this is where you add any other method you might need.
<details>
 <summary>Expand Boilerplate</summary>

```

import api from "./apiConfig";

export const getVariables = async () => {
try {
const response = await api.get("/variables");
return response.data;
} catch (error) {
throw error;
}
};

export const getVariable = async (id) => {
try {
const response = await api.get(`/variables/${id}`);
return response.data;
} catch (error) {
throw error;
}
};

export const createVariable = async (variable) => {
try {
const response = await api.post("/variables", variable);
return response.data;
} catch (error) {
throw error;
}
};

export const updateVariable = async (id, variable) => {
try {
const response = await api.put(`/variables/${id}`, variable);
return response.data;
} catch (error) {
throw error;
}
};

export const deleteVariable = async (id) => {
try {
const response = await api.delete(`/variables/${id}`);
return response.data;
} catch (error) {
throw error;
}
};

```
</details>
</details>

# Common Errors w/Troubleshooting<a name="errors"></a>
# CSS Rules / Tricks<a name="css"></a>
# React<a name="react"></a>
```
