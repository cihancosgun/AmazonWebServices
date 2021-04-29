 # Amazon Amplify Expo React Native Mobil Uygulama Geliştirme Çalışması

Orijinal Tutorial İçin : https://docs.amplify.aws/start/getting-started/installation/q/integration/react-native

## Amplify Nedir?
Amazon'un mobil uygulama geliştiriciler için sunmuş olduğu bir platformdur. Bu platform sayesinde mobil uygulamaların backend tarafını serverless mimari üzerine kurabiliyoruz. Bu servis içerisinde Amazon Cognito ile kullanıcı kayıt ve yetkilendirmelerimizi, Amazon Lambda ile backend fonksiyonlarını, Amazon DynamoDB de verilerimizi tutacağız, Amazon S3 ile dosyalarımızı saklayabileceğiz.

### 1. Amplify Cli Kurulumu
Amplify Cli, mobil uygulama için gerekli konfigurasyonları bizim için otomatik yapan bir komut satırı arayüzüdür (Command Line Interface).
```
sudo npm install -g @aws-amplify/cli
```
### 2. Amplify Konfigurasyon
Aşağıdaki komutu ile, bundan sonra amplify için gerekli olan konfigurasyonu yapıyoruz.
```
amplify configure
```
Komut sizi otomatik olarak AWS konsola yönlendirecek, sırasıyla kullanıcı oluşturma ve rol tanımlamaları yapmanızı isteyecek. Mobil uygulamanızın backend servisleri için bir IAM kullanıcısı oluşturun ve `AdministratorAccess` yektisi verin. Son aşamada accessKeyId ve secretAccessKey bilgilerini konsol ekranına girin.

 ### 3. Expo ile React Native Uygulamamızı Oluşturalım
 ```console
 npm install -g expo-cli 
 expo init RNAmplify 

? Choose a template: blank

cd RNAmplify
amplify init
npm install aws-amplify aws-amplify-react-native @react-native-community/netinfo
 ```
### 4. App.js Dosyasını Düzenleyin
```javascript
import Amplify from 'aws-amplify' 
import config from './src/aws-exports' 
Amplify.configure(config)
```
### 5. GraphQL Api ve DB Oluşturma
```console
amplify add api #accept defaults
```
Seçimler:
```
? Please select from one of the below mentioned services:
#GraphQL
? Provide API name:
#(myapi)
? Choose the default authorization type for the API:
#API Key
? Enter a description for the API key:
#demo
? After how many days from now the API key should expire:
#7 (or your preferred expiration)
? Do you want to configure advanced settings for the GraphQL API:
#No
? Do you have an annotated GraphQL schema?
#No
? Do you want a guided schema creation?
#Yes
? What best describes your project:
#Single object with fields
? Do you want to edit the schema now?
#No
 ```
 ### 6. Deploy
 ```
 amplify push
 ```
 Seçimler:
 ```
? Are you sure you want to continue? Y
#If you did not mock the API, you will be walked through the following questions for GraphQL code generation
? Do you want to generate code for your newly created GraphQL API? Y
? Choose the code generation language target: javascript
? Do you want to generate/update all possible GraphQL operations - queries, mutations and subscriptions? Y
? Enter maximum statement depth [increase from default if your schema is deeply nested]: 2
 ```
### 7. Kullanıcı Giriş - Kayıt Ekranları Ekleme (Auth)
```
amplify add auth
```
Seçimler :
```
? Do you want to use the default authentication and security configuration? Default configuration
? How do you want users to be able to sign in? Username
? Do you want to configure advanced settings?  No, I am done.
```
```
amplify push

npm install aws-amplify aws-amplify-react-native @react-native-community/netinfo

```
#### App.js Düzenleme
```javascript
import React, { useEffect, useState } from 'react'
import {
  View, Text, StyleSheet, TextInput, Button
} from 'react-native'

import { API, graphqlOperation } from 'aws-amplify'
import { createTodo } from './src/graphql/mutations'
import { listTodos } from './src/graphql/queries'
// Kullanıcı Yetkilendirme Modülü (Auth)
import { withAuthenticator } from 'aws-amplify-react-native'

import Amplify from 'aws-amplify'
import config from './src/aws-exports'
//AWS Konfigurasyon Dosyası
Amplify.configure(config)

const initialState = { name: '', description: '' }

const App = () => {
  const [formState, setFormState] = useState(initialState)
  const [todos, setTodos] = useState([])

  useEffect(() => {
    fetchTodos()
  }, [])

  function setInput(key, value) {
    setFormState({ ...formState, [key]: value })
  }

  async function fetchTodos() {
    try {
      const todoData = await API.graphql(graphqlOperation(listTodos))
      const todos = todoData.data.listTodos.items
      setTodos(todos)
    } catch (err) { console.log('error fetching todos') }
  }

  async function addTodo() {
    try {
      const todo = { ...formState }
      setTodos([...todos, todo])
      setFormState(initialState)
      await API.graphql(graphqlOperation(createTodo, {input: todo}))
    } catch (err) {
      console.log('error creating todo:', err)
    }
  }

  return (
    <View style={styles.container}>
      <TextInput
        onChangeText={val => setInput('name', val)}
        style={styles.input}
        value={formState.name}
        placeholder="Name"
      />
      <TextInput
        onChangeText={val => setInput('description', val)}
        style={styles.input}
        value={formState.description}
        placeholder="Description"
      />
      <Button title="Create Todo" onPress={addTodo} />
      {
        todos.map((todo, index) => (
          <View key={todo.id ? todo.id : index} style={styles.todo}>
            <Text style={styles.todoName}>{todo.name}</Text>
            <Text>{todo.description}</Text>
          </View>
        ))
      }
    </View>
  )
}

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: 'center', padding: 20 },
  todo: {  marginBottom: 15 },
  input: { height: 50, backgroundColor: '#ddd', marginBottom: 10, padding: 8 },
  todoName: { fontSize: 18 }
})

export default withAuthenticator(App)
```
#### Test Edebiliriz
```
EXPO_LEGACY_IMPORTS=1 expo start
```
