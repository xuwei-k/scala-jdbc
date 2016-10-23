# scala-jdbc

Better JDBC wrapper for Scala.

```scala
libraryDependencies += "com.github.takezoe" %% "scala-jdbc" % "1.0.2"
```

You can use better-jdbc by adding a following import statements:

```scala
import com.github.takezoe.scala.jdbc._
```

## Select

Extract values from `ResultSet`:

```scala
val users: Seq[(Int, String)] = DB.autoClose(conn) { db =>
  db.select("SELECT * FROM USERS"){ rs =>
    (rs.getInt("USER_ID"), rs.getString("USER_NAME"))
  }
}
```

Retrieve a first record:

```scala
val user: Option[(Int, String)] = DB.autoClose(conn) { db =>
  db.selectFirst(sql"SELECT * FROM USERS WHERE USER_ID = $userId"){ rs =>
    (rs.getInt("USER_ID"), rs.getString("USER_NAME"))
  }
}
```

Map `ResultSet` to the case class:

```scala
case class User(userId: Int, userName: String)

val users: Seq[User] = DB.autoClose(conn) { db =>
  db.select("SELECT USER_ID, USER_NAME FROM USERS", User.apply)
}
```

## Update

```scala
DB.autoClose(conn) { db =>
  db.update(sql"INSERT INTO USERS (USER_ID, USER_NAME) VALUES ($userId, $userName)")
}
```

## Transaction

```scala
DB.autoClose(conn) { db =>
  db.transaction {
    db.update(sql"DELETE FROM GROUP WHERE USER_ID = $userId")
    db.update(sql"DELETE FROM USERS WHERE USER_ID = $userId")
  }
}
```

## Large data

Process large data using `scan` method:

```scala
DB.autoClose(conn) { db =>
  db.scan("SELECT * FROM USERS"){ rs =>
    println(rs.getString("USER_NAME"))
  }
}
```

## SQL Validation (Experimental)

scala-jdbc provides `sqlc` macro that validates a given SQL. You can use it instead of sql string interpolation.

```scala
db.selectFirst(sqlc("SELECT * FROM USERS WHERE USER_ID = $userId")){ rs =>
  (rs.getInt("USER_ID"), rs.getString("USER_NAME"))
}
```

When a given SQL is invalid, this macro reports error in compile time.

In default, this macro checks only sql syntax using [JsqlParser](https://github.com/JSQLParser/JSqlParser).

It's also possible to validate by throwing SQL against the actual database.
Create `database.json` in the current directory with following content to enable this feature:


```javascript
{
  "driver": "org.h2.Driver",
  "url": "jdbc:h2:~/.gitbucket/data",
  "user": "sa",
  "password": "sa"
}
```

However `sqlc` macro is still experimental feature, so you have to use SNAPSHOT version of scala-jdbc to use it:

```scala
resolvers += "Sonatype OSS Snapshots" at "https://oss.sonatype.org/content/repositories/snapshots"

libraryDependencies += "com.github.takezoe" %% "scala-jdbc" % "1.0.3-SNAPSHOT"
```

If you get invalid validation results, please report them to [issues](https://github.com/takezoe/scala-jdbc/issues).


