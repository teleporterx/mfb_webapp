# Database Setup

Since I'm using **MongoDB** with the Motor driver in an asynchronous environment, there’s no need for **traditional database migrations** like you would find in relational databases (such as SQL). MongoDB's schema-less nature gives me the flexibility to create and modify collections **on-the-fly** without needing strict migrations. This allows me to **dynamically adjust the database structure** as the **application evolves**.

Heres a handy [Guide](https://docs.mongoengine.org/guide/migration.html) that talks about Documents migration in Mongo.

Additionally, for alterations or manual modifications, I can use tools like the **Mongo Shell** or **MongoDB Compass** to interact with and modify the database as needed. These tools provide a simple way to manage collections, indexes, and documents directly.

> MongoDB has been chosen as the database for this project because of its **simplicity** and **flexibility**.



**MongoDB Service Configuration Details**
|Key|Result|
|-|-|
|**Database**|MongoDB|
|**Database Type**|NoSQL|
|**Driver**|Motor (asynchronous MongoDB driver)|
|**Access Method**|Asynchronous|
|**Service File Location**|`/api/v1/services/mongo.py`|
|**Deployment Type**|Docker|

<br>
<br>

**Docker Deployment**
```bash
# Container creation
sudo docker run -d --name mongoose -p 27017:27017 mongo:latest
# Container reuse
sudo docker start|stop mongoose
```


## Description:
`MongoDB` class a reusable object that facilitates **asynchronous interaction** with any **MongoDB database** using the `motor` library has been coded for **simplifying database operations**. It provides methods to:

- Connect to a MongoDB server.
- Perform CRUD operations: insert, find, update, and delete documents.
- Create indexes on specified fields.
- Close the MongoDB client connection.

The class utilizes `AsyncIOMotorClient` for asynchronous database access, supporting operations like inserting, querying, and updating data, while ensuring non-blocking I/O operations.

## MongoDB Compass:

- For checking DB schema and data we can use tools like mongosh.
- MongoDB compass offers a GUI interface for managing MongoDB databases which is more intuitive and user-friendly for altering/validating the database structure on the go.

![MongoDB Compass](assets/mongodb_compass.png)

## Implementation:

Refer the docstrings for implementation details.

```python
# /api/v1/services/mongo.py

from motor.motor_asyncio import AsyncIOMotorClient
from bson import ObjectId
from typing import List, Dict, Any, Optional


class MongoDB:
    def __init__(self, uri: str):
        """
        Initialize a MongoDB instance with the given URI.

        This constructor creates an AsyncIOMotorClient instance using the provided URI,
        establishing a connection to the MongoDB server.

        Args:
            uri (str): The MongoDB connection string URI.

        Returns:
            None

        Note:
            This method initializes the `client` attribute of the class instance,
            which is used for subsequent database operations.
        """
        self.client = AsyncIOMotorClient(uri)

    def get_collection(self, db_name: str, collection_name: str):
        """
        Get a MongoDB collection from a specified database.

        This method retrieves a collection object from the specified database
        using the provided database name and collection name.

        Args:
            db_name (str): The name of the database.
            collection_name (str): The name of the collection within the database.

        Returns:
            Collection: A MongoDB collection object that can be used for database operations.
        """
        db = self.client[db_name]
        return db[collection_name]

    async def insert_one(self, db_name: str, collection_name: str, data: Dict[str, Any]) -> str:
        """
        Insert a single document into a specified MongoDB collection.

        This asynchronous method inserts one document into the specified collection
        of the given database.

        Args:
            db_name (str): The name of the database.
            collection_name (str): The name of the collection within the database.
            data (Dict[str, Any]): A dictionary representing the document to be inserted.

        Returns:
            str: The string representation of the inserted document's ObjectId.

        """
        collection = self.get_collection(db_name, collection_name)
        result = await collection.insert_one(data)
        return str(result.inserted_id)

    async def insert_many(self, db_name: str, collection_name: str, data_list: List[Dict[str, Any]]) -> List[str]:
        """
        Insert multiple documents into a specified MongoDB collection.

        This asynchronous method inserts multiple documents into the specified collection
        of the given database.

        Args:
            db_name (str): The name of the database.
            collection_name (str): The name of the collection within the database.
            data_list (List[Dict[str, Any]]): A list of dictionaries, each representing a document to be inserted.

        Returns:
            List[str]: A list of string representations of the inserted documents' ObjectIds.
        """
        collection = self.get_collection(db_name, collection_name)
        result = await collection.insert_many(data_list)
        return [str(id) for id in result.inserted_ids]

    async def find_one(self, db_name: str, collection_name: str, query: Dict[str, Any]) -> Optional[Dict[str, Any]]:
        """
        Find and return a single document from a specified MongoDB collection based on the given query.

        This asynchronous method finds and returns a single document from the specified collection
        of the given database based on the provided query. If the '_id' field is a string, it is converted
        to an ObjectId before executing the query.

        Args:
            db_name (str): The name of the database.
            collection_name (str): The name of the collection within the database.
            query (Dict[str, Any]): A dictionary representing the query to be executed.

        Returns:
            Optional[Dict[str, Any]]: An optional dictionary representing the found document. If no document is found,
            this method returns None. The '_id' field of the returned document is converted to a string.
        """
        collection = self.get_collection(db_name, collection_name)
        if '_id' in query and isinstance(query['_id'], str):
            query['_id'] = ObjectId(query['_id'])
        document = await collection.find_one(query)
        if document:
            # Convert ObjectId to string
            document['_id'] = str(document['_id'])
        return document

    async def find_all(self, db_name: str, collection_name: str, query: Dict[str, Any] = {}) -> List[Dict[str, Any]]:
        """
        Find and return all documents from a specified MongoDB collection based on the given query.

        This asynchronous method finds and returns all documents from the specified collection
        of the given database based on the provided query. If the '_id' field is a string, it is converted
        to an ObjectId before executing the query.

        Args:
            db_name (str): The name of the database.
            collection_name (str): The name of the collection within the database.
            query (Dict[str, Any], optional): A dictionary representing the query to be executed. Defaults to an empty dictionary.

        Returns:
            List[Dict[str, Any]]: A list of dictionaries representing the found documents. If no documents are found,
            this method returns an empty list. The '_id' field of each returned document is converted to a string.
        """
        collection = self.get_collection(db_name, collection_name)
        documents = []
        if '_id' in query and isinstance(query['_id'], str):
            query['_id'] = ObjectId(query['_id'])
        async for document in collection.find(query):
            # Convert ObjectId to string
            document['_id'] = str(document['_id'])
            documents.append(document)
        return documents

    async def update_one(self, db_name: str, collection_name: str, query: Dict[str, Any], update_data: Dict[str, Any]) -> int:
        """
        Update a single document in a specified MongoDB collection based on the given query.

        This asynchronous method updates a single document in the specified collection
        of the given database based on the provided query and update data. If the '_id' field
        in the query is a string, it is converted to an ObjectId before executing the update.

        Args:
            db_name (str): The name of the database.
            collection_name (str): The name of the collection within the database.
            query (Dict[str, Any]): A dictionary representing the query to find the document to update.
            update_data (Dict[str, Any]): A dictionary representing the data to update in the document.

        Returns:
            int: The number of documents modified. In this case, it will be either 0 or 1,
            indicating whether the update operation was successful.
        """
        collection = self.get_collection(db_name, collection_name)
        if '_id' in query and isinstance(query['_id'], str):
            query['_id'] = ObjectId(query['_id'])
        result = await collection.update_one(query, {'$set': update_data})
        return result.modified_count

    async def delete_one(self, db_name: str, collection_name: str, query: Dict[str, Any]) -> int:
        """
        Delete a single document from a specified MongoDB collection based on the given query.

        This asynchronous method deletes a single document from the specified collection
        of the given database based on the provided query. If the '_id' field in the query
        is a string, it is converted to an ObjectId before executing the deletion.

        Args:
            db_name (str): The name of the database.
            collection_name (str): The name of the collection within the database.
            query (Dict[str, Any]): A dictionary representing the query to find the document to delete.

        Returns:
            int: The number of documents deleted. In this case, it will be either 0 or 1,
            indicating whether the deletion operation was successful.
        """
        collection = self.get_collection(db_name, collection_name)
        if '_id' in query and isinstance(query['_id'], str):
            query['_id'] = ObjectId(query['_id'])
        result = await collection.delete_one(query)
        return result.deleted_count

    async def create_index(self, db_name: str, collection_name: str, field_name: str, unique: bool = False) -> int:
        """
        Create an index on a specified field in a MongoDB collection.

        This asynchronous method creates an index on the specified field in the given collection
        of the specified database. If the index is unique, duplicate values are not allowed.

        Args:
            db_name (str): The name of the database.
            collection_name (str): The name of the collection within the database.
            field_name (str): The name of the field on which to create the index.
            unique (bool, optional): A flag indicating whether the index should be unique. Defaults to False.

        Returns:
            int: The result of the index creation operation. If the index creation is successful,
            this method returns 1. Otherwise, it returns 0.
        """
        collection = self.get_collection(db_name, collection_name)
        result = await collection.create_index([(field_name, 1)], unique=unique)
        return result

    def close(self):
        """
        Close the MongoDB client connection.

        This method closes the connection to the MongoDB server by calling the `close()` method
        of the `AsyncIOMotorClient` instance. This method should be called when the MongoDB instance
        is no longer needed to free up system resources.

        Parameters:
            None

        Returns:
            None
        """
        self.client.close()
```