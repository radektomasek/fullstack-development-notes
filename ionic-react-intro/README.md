# Introduction to Ionic React

This is a simple overview of the capabilities of Ionic Framework using React.

## Using Ionic without a framework

Initially, Ionic was created working with Angular, that utilized the web-components
and helped with the innovation on the frontend landscape. Although, we are going to
show some practical examples in React and TypeScript, in reality, you can use any
major framework on no framework at all. Let's have a quick look of how to initialize a
project with no framework.

Probably the easiest way is to follow the CDN section in the documentation:
https://ionicframework.com/docs/intro/cdn and copy the following snippet to your html.

```html
<script type="module"
src="https://cdn.jsdelivr.net/npm/@ionic/core/dist/ionic/ionic.esm.js"></script>
<script nomodule src="https://cdn.jsdelivr.net/npm/@ionic/core/dist/ionic/ionic.js">
</script>
<link rel="stylesheet"
href="https://cdn.jsdelivr.net/npm/@ionic/core/css/ionic.bundle.css"/>
```
We might also add a few initial components. In this situation, these are just dummy
ones to demonstrate the main benefit of using Ionic over anything else - adaptivity
based on the platform (iOS, Android or web). We will also cover basics of Ionic grid
system.

List of components we are going to add (all of them have many ways of how to tweak
params, but we really scratch a surface):

```
ion-grid: https://ionicframework.com/docs/api/grid
ion-col: https://ionicframework.com/docs/api/col
ion-row: https://ionicframework.com/docs/api/row
ion-toggle: https://ionicframework.com/docs/api/toggle
ion-button: https://ionicframework.com/docs/api/button
```

> Note: In this very specific case, it really doesn't matter much in which order you put these components.

A complete version of the initial page might look as the following:

```html
<!-- index.html --->
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Ionic</title>
<script type="module"
src="https://cdn.jsdelivr.net/npm/@ionic/core/dist/ionic/ionic.esm.js"></script>
<script nomodule src="https://cdn.jsdelivr.net/npm/@ionic/core/dist/ionic/ionic.js">
</script>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@ionic/core/css/ionic.bundle.css"/>
</head>
<body>
    <ion-grid>
        <ion-row>
            <ion-col>
                <ion-toggle></ion-toggle>
            </ion-col>
        </ion-row>
        <ion-row>
            <ion-col>
                <ion-button>Default</ion-button>
            </ion-col>
            <ion-col>
                <ion-button color="secondary">Secondary</ion-button>
            </ion-col>
        </ion-row>
    </ion-grid>
</body>
</html>
```
## Using Ionic with React

Probably the easiest way how to integrate an Ionic into a React app is just to include
a few packages and some extra styling. This document tells you more about it.

It might be useful, in case you are working with an existing web project/start
prototyping a native UI and you just want to include Ionic web-components.

However, if you start from scratch, it is better to use @ionic/cli that adds
everything you might need and simplifies the whole process. It also adds support of
TypeScript by default.


> Note: Although docs suggests installing the cli tool globally, I recommend to
stick to the npx option. An advantage of this option is that it always fetches the
latest version from the web.

## Create an empty project

To create a new project using ionic/cli, just type the following command:

```bash
npx @ionic/cli start my-notes blank --type=react --capacitor
```

> Note: This will create an empty (blank) project based on React template. You can
also set different starting layouts. Checkout the documentation for more
information.

## Starting files

In this sections, we are going to put some helper files (type definition, mocks and
api), which are going to help us during the implementation. They should be pretty
self-explanatory and are all fully based on my previous workshops (e.g. Vue Notes
App).

Let's create three folders utils, mocks and types and add an empty index.ts file
to each of them. And then, let's add the following code:

### types/index.ts

```typescript
// types/index.ts
export type NoteColor = 'red' | 'green' | 'orange' | '';

export interface Note {
id: string;
title: string;
description: string;
color: NoteColor;
}
```

### utils/index.ts
  
```typescript
// utils/index.ts
import type { NoteColor } from '../types';

export const getColorTheme = (color: NoteColor): string => {
  switch (color) {
    case 'green':
      return 'success';
    case 'red':
      return 'danger';
    case 'orange':
      return 'warning';
    default:
      return 'primary';
  }
};
```
### mocks/index.ts

```typescript
// mocks/index.ts
import type { Note } from '../types';

export const mockNotes: Note[] = [
  {
    id: 'pktnj',
    title: 'My first note',
    description: 'This is a description of the first note',
    color: 'green',
  },
  {
    id: 'oz98cm',
    title: 'Second note',
    description: 'This is a description of the second note',
    color: 'red',
  },
  {
    id: 'b0125m',
    title: 'Third note',
    description: 'This is a description of the third note',
    color: 'orange',
  },
]
```

### App.tsx

```typescript
// App.tsx
import { Redirect, Route } from 'react-router-dom';
import { IonApp, IonRouterOutlet } from '@ionic/react';
import { IonReactRouter } from '@ionic/react-router';
import { Auth } from './pages/Auth';
import { NotesTabs } from './pages/NotesTabs';

const App: React.FC = () => (
  <IonApp>
    <IonReactRouter>
      <IonRouterOutlet>
        <Route exact path="/login" >
          <Auth />
        </Route>
        <Route path="/notes">
          <NotesTabs />
        </Route> 
        <Route exact path="/">
          <Redirect to="/login" />
        </Route> 
      </IonRouterOutlet>
    </IonReactRouter> 
  </IonApp>
);

export default App;
```


### pages/Auth.tsx

```typescript
// Auth.tsx
import React from 'react';
import {
    IonPage,
    IonHeader,
    IonContent,
    IonToolbar,
    IonTitle,
    IonItem,
    IonLabel,
    IonInput,
    IonButton,
    IonGrid,
    IonRow,
    IonCol,
    IonAlert,
} from '@ionic/react';
import { useHistory } from 'react-router-dom';

export const Auth: React.FC = () => {
  // const usernameRef = React.useRef<HTMLIonInputElement>(null);
  // const passwordRef = React.useRef<HTMLIonInputElement>(null);
  const [username, setUsername] = React.useState<string>('');
  const [password, setPassword] = React.useState<string>('');
  const [error, setError] = React.useState<string>('');
  const history = useHistory();
    
  const onUsernameChange = (event: CustomEvent) => {
    event.preventDefault();
    setUsername(event.detail.value);
  };
    
  const onPasswordChange = (event: CustomEvent) => {
    event.preventDefault();
    setPassword(event.detail.value);
  };
    
  const clearError = () => {
    setError('');
  };
    
  const handleLogin = () => {
    if (
      !username || !password || username.trim().length === 0 || password.trim().length === 0
    ) {
      setError('Please enter a username and password');
      return;
    }
    
    history.push('/notes');
  };
    
    return (
      <IonPage>
        <IonAlert
          isOpen={Boolean(error)}
          message={error}
          buttons={[
            {
              text: 'Okay',
              handler: clearError,
            },
          ]}
        />
        
        <IonHeader>
          <IonToolbar color="primary">
            <IonTitle className="ion-text-center">Login Page</IonTitle>
          </IonToolbar>
        </IonHeader>
        <IonContent className="ion-padding">
          <IonGrid>
            <IonRow>
              <IonCol>
                <IonItem>
                  <IonLabel position="floating">Username</IonLabel>
                  <IonInput
                    name="username"
                    type="text"
                    value={username}
                    onIonChange={onUsernameChange}
                  />
                </IonItem>
              </IonCol>
            </IonRow>
          <IonRow>
            <IonCol>
              <IonItem>
                <IonLabel position="floating">Password</IonLabel>
                <IonInput
                    name="password"
                    type="password"
                    value={password}
                    onIonChange={onPasswordChange}
                />
              </IonItem>
            </IonCol>
          </IonRow>
          <IonRow className="ion-text-center">
            <IonCol>
              <IonButton onClick={handleLogin}>Login</IonButton>
            </IonCol>
          </IonRow>
        </IonGrid>
      </IonContent>
    </IonPage>
  );
};
```
### pages/NoteDetail.tsx

```typescript
// NoteDetail.tsx
import React from 'react';
import {
  IonHeader,
  IonContent,
  IonToolbar,
  IonTitle,
  IonPage,
  IonButtons,
  IonBackButton,
  IonCard,
  IonCardContent,
  IonCardHeader,
  IonCardTitle,
  IonCardSubtitle,
} from '@ionic/react';
import { useParams } from 'react-router-dom';
import { mockNotes } from '../mocks';
import { getColorTheme } from '../utils';
import type { Note } from '../types';

export const NoteDetail: React.FC = () => {
  const { id } = useParams<{ id: string }>();
  const [note, setNote] = React.useState<Note | null>(null);
    
  React.useEffect(() => {
    const note = mockNotes.find((note) => note.id === id);
    if (note) {
    setNote(note);
    }
  }, []);
    
  return (
    <IonPage>
      <IonHeader>
        <IonToolbar>
          <IonButtons slot="start">
            <IonBackButton defaultHref="/notes/list" />
          </IonButtons>
          <IonTitle>Note Detail</IonTitle>
        </IonToolbar>
      </IonHeader>
      <IonContent>
      {!note && (
        <IonCard color="light">
          <IonCardHeader>
            <IonCardTitle>Not found</IonCardTitle>
          </IonCardHeader>
          <IonCardContent>
            <div>
              Record with id: <strong>{id}</strong> wasn't found!
            </div>
          </IonCardContent>
        </IonCard>
      )}
      {note && (
        <IonCard color={getColorTheme(note.color)}>
          <IonCardHeader>
            <IonCardTitle>{note.title}</IonCardTitle>
            <IonCardSubtitle>{note.color}</IonCardSubtitle>
          </IonCardHeader>
          <IonCardContent>
            <div className="ion-text-left">{note.description}</div>
          </IonCardContent>
        </IonCard>
      )}
    </IonContent>
  </IonPage>
  );
};
```


### pages/NotesList.tsx

```typescript
// NotesList.tsx
import React from 'react';
import {
  IonHeader,
  IonContent,
  IonToolbar,
  IonTitle,
  IonPage,
  IonList,
  IonItem,
  IonLabel,
} from '@ionic/react';
import { mockNotes } from '../mocks';
import type { Note } from '../types';

export const NotesList: React.FC = () => {
  const [notes, setNotes] = React.useState<Note[]>([]);

  React.useEffect(() => {
    setNotes(mockNotes);
  }, []);
  
  return (
    <IonPage>
      <IonHeader>
        <IonToolbar>
          <IonTitle>Notes</IonTitle>
        </IonToolbar>
      </IonHeader>
      <IonContent>
        <IonList>
        {notes.map((note) => (
          <IonItem key={note.id} button href={`/notes/${note.id}`}>
            <IonLabel>
              <h2>{note.title}</h2>
            </IonLabel>
          </IonItem>
        ))}
        </IonList>
      </IonContent>
    </IonPage>
  );
};
```
### pages/NotesTabs.tsx

```typescript
// NoteTabs.tsx
import React from 'react';
import {
  IonTabs,
  IonRouterOutlet,
  IonTabBar,
  IonIcon,
  IonLabel,
  IonTabButton,
} from '@ionic/react';
import { Route, Redirect, Switch } from 'react-router-dom';
import { NotesList } from './NotesList';
import { NoteDetail } from './NoteDetail';
import { Settings } from './Settings';
import { settingsOutline, listOutline } from 'ionicons/icons';

export const NotesTabs: React.FC = () => {
  return (
    <IonTabs>
      <IonRouterOutlet>
        <Redirect path="/notes" to="/notes/list" exact />
        <Switch>
          <Route path="/notes/settings" exact>
            <Settings />
          </Route>
          <Route path="/notes/list" exact>
            <NotesList />
          </Route>
          <Route path="/notes/:id">
            <NoteDetail />
          </Route>
        </Switch>
      </IonRouterOutlet>
      <IonTabBar slot="bottom">
        <IonTabButton tab="notes" href="/notes/list">
          <IonIcon icon={listOutline} />
          <IonLabel>Notes</IonLabel>
        </IonTabButton>
        <IonTabButton tab="settings" href="/notes/settings">
          <IonIcon icon={settingsOutline} />
          <IonLabel>Settings</IonLabel>
        </IonTabButton>
      </IonTabBar>
    </IonTabs>
  );
};
```

### pages/Setings.tsx

```typescript
// Settings.tsx
import React from 'react';
import {
  IonHeader,
  IonContent,
  IonToolbar,
  IonTitle,
  IonPage,
  IonButton,
  IonText,
  isPlatform,
  IonIcon,
} from '@ionic/react';
import { logoAndroid, logoApple } from 'ionicons/icons';
import { useHistory } from 'react-router-dom';

export const Settings: React.FC = () => {
  const history = useHistory();
  const handleLogout = () => {
  history.replace('/');
};
  
return (
  <IonPage>
    <IonHeader>
      <IonToolbar>
        <IonTitle>Settings</IonTitle>
      </IonToolbar>
    </IonHeader>
    <IonContent className="ion-padding">
    {isPlatform('android') && (
      <IonText>
        Your platform is <IonIcon icon={logoAndroid} />
      </IonText>
    )}
    {isPlatform('ios') && (
      <IonText>
        Your platform is <IonIcon icon={logoApple} />
      </IonText>
    )}
    <IonButton onClick={handleLogout} expand="full">
      Logout
    </IonButton>
    </IonContent>
  </IonPage>
  );
};
```
## Open in emulator (native app)

We can open the app in the emulator or prepare a package for native app.

The first step is to build a project:

```bash
npm run build
```
When we initialized the project, we already added a capacitor. But we also need to
specify the platform.

```bash
npx cap add android # or ios
npx cap sync
```
To open the project in Android Studio/emulator (or iOS Similator), you can type:

```bash
npx cap open android # or ios
```
## Conclusion

Thanks for following my notes. Any feedback appreciated (radek.tomasek@gmail.com).