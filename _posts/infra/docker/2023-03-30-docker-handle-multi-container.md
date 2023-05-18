---
author: winverse
title:  "[Docker] 2.다중 컨테이너 애플리케이션 구축하기"
tags: 
  - docker
categories: 
  - infra
is_published: true
show_date: true
date: "2023-03-30"
description: "Docker를 이용한 다중 컨테이너 애플리케이션을 어떻게 구축하는지 알아봅시다."
---

# 1. 다중 컨테이너를 운영하기 위해서 필수로 알아야하는 지식
1. Image
2. Container
3. Volume
4. Network

# 2. 다중 컨테이너 운영하기 위한 Code
코드가 중요한 것은 아니기에 중요 부분을 제외한 코드들 제외하였습니다.

## 2-1. Frontend 
리액트를 이용하였으며 일부분입니다.
```javascript
// app.js
function App() {
  const [loadedGoals, setLoadedGoals] = useState([]);
  const [error, setError] = useState(null);

  useEffect(function () {
    async function fetchData() {
      try {
        const response = await fetch("http://localhost:80/goals");

        const resData = await response.json();

        if (!response.ok) {
          throw new Error(resData.message || "Fetching the goals failed.");
        }

        setLoadedGoals(resData.goals);
      } catch (err) {
        setError(
          err.message ||
            "Fetching goals failed - the server responsed with an error."
        );
      }
    }

    fetchData();
  }, []);

  return (
    <div>
      {error && <ErrorAlert errorText={error} />}
      <GoalInput />
      {!isLoading && (
        <CourseGoals goals={loadedGoals}/> // list
      )}
    </div>
  );
}
```
## 2-2. Backend
Nodejs를 이용했으며 일부분입니다.
```javascript
const app = express();

app.get("/goals", async (req, res) => {
  try {
    const goals = await Goal.find();
    res.status(200).json({
      goals: goals.map((goal) => ({
        id: goal.id,
        text: goal.text,
      })),
    });
  } catch (err) {
    console.error(err.message);
    res.status(500).json({ message: "Failed to load goals." });
  }
});

mongoose.connect(
  `mongodb://localhost:27017/course-goals`,
  {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  },
  (err) => {
    if (err) {
      console.error("FAILED TO CONNECT TO MONGODB");
      console.error(err);
    } else {
      console.log("CONNECTED TO MONGODB");
      app.listen(80);
    }
  }
);
```

## 2-3. Mongodb
docker hub에서 image를 가져옵니다.
```sh
docker pull mongo
```

# 3. Build & Container 만들기
Dockerfile은 [이전 글](/infra/2023/03/30/docker-basic-command.html)을 참고해주세요!

## 3-1. Create Image
```sh
# build for front
docker build -t front .

# build for back
docker build -t back .

# build for database
docker images 
```
## 3-2. Create Container
```sh
# front
docker run -it --rm -d -p 3000:3000 --name front front

# back
docker run -it --rm -d -p 80:80 --name back back

# mongo
docker run -rm -d -p 27017:27017 --name mongo mongo
```

# 4. 문제 발생
이렇게 각 컨테이너를 따로 생성하게 되면 때문에 back 내부 container에서는 mongoDB를 인식할 수 없게 됩니다. 그래서 `localhost:27017` 부분을 `host.docker.internal:27017`으로 수정합니다.

## 4-1. host.docker.internal
host.docker.internal은 Docker for Windows와 Docker for Mac에서 호스트 시스템의 IP 주소를 참조하는 데 사용되는 특수 DNS 이름입니다. 이 이름을 사용하면 컨테이너에서 호스트 시스템의 서비스에 접근할 수 있습니다. 하지만 이 방법은 Docker for Windows와 Docker for Mac에서만 작동하며 Linux에서는 작동하지 않습니다. Linux에서는 다른 방법을 사용해야 합니다.

```javascript
# 주소 수정
mongoose.connect(
  "mongodb://host.docker.internal:27017/course-goals",
  {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  },
);
```
그런데 이렇게 하게 되면 또한 로컬에서는 작동 시킬 수 없는 단점이 있습니다. 

# 5. Network 활용 해보기

## 5-1. Create Network
```sh
docker network create my-network
```

## 5-2. Create Container
```sh
# front
# network 옵션이 없음
docker run -it --rm -d -p 3000:3000 --name front front

# back
# front와 통신 할 port 번호가 필요함
dcoker run -it --rm -d -p 80:80 --network my-network --name back back

# mongo
# 같은 network를 사용하기 때문에 포트 번호가 필요가 없음
# 여기에 volume 플래그를 추가하면 데이터를 지속할 수 있습니다
docker run -rm -d --network my-network --name mongo mongo
```
여기서 주의 할 점은 front는 network로 묶어도 의미 없다는 것과 back&database은 하나의 network로 묶어두되 back은 front와 통신할 port가 필요하다는 것 입니다.

front 코드는 결국 browser에서 작동하기 때문에 docker network를 지정해준다고 해도 browser가 인식하지 못하기 때문에 의미가 없습니다.

## 5-3. 코드 수정
```js
# back에서 mongodb와 connect 하기 위해서는 컨테이너 이름이 들어가야 한다.
const mongoContainerName = 'mongo';
mongoose.connect(
  `mongodb://${mongoContainerName}:27017/course-goals`,
  {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  },
);

# 혹은 .env를 이용하여 주소값에 username과 password를 입력하는 방법도 있다.
mongoose.connect(
  `mongodb://${process.env.MONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@mongodb:27017/course-goals`,
  {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  },
);
```

# 6. 애플리케이션을 동작 시키기 위한 Command 정리
다음은 3개의 어플리케이션을 동작시키기 위해 결국 프로그래머가 shell에 입력해야하는 코드를 알아보겠습니다.

```sh
# Create Network
docker network create my-network

# Craete Volume
docker volume create data

# Build
docker build -t front .
docker build -t back .
docker build -t mongo .

# Run React SPA Container
docker run --name react-front \
  -v ${pwd}/src:/app/src \
  --rm \
  -d \
  -p 3000:3000 \
  -it \
  front

# Run Node API Container
docker run --name node-back \
  -e MONGODB_USERNAME=max \
  -e MONGODB_PASSWORD=secret \
  # Named volume
  -v logs:/app/logs \ 
  # Bind Mounts Volume
  -v ${pwd}/src:/app \
  # Anonymouse Volume
  -v /app/node_modules \
  --rm \
  -d \
  --network my-network \
  -p 80:80 \
  back

# Run MongoDB Container
docker run --name mongodb \
  -e MONGO_INITDB_ROOT_USERNAME=usename \
  -e MONGO_INITDB_ROOT_PASSWORD=secret \
  -v data:/data/db \ 
  --rm \
  -d \
  --network my-network \
  mongo


# Stop all Containers
docker stop mongodb node-back react-front
```

# 7. 정리
다중 애플리케이션을 작동시키는 방법을 간략하게 알아보았으며 이를 통해서 Docker를 통한 개발과 여러 flag들을 다뤄보며 익숙해지는 시간을 가졌습니다. 그러나 애플리케이션을 동작시키기 위한 command가 굉장히 길어지는 것을 알 수 있는데요. 이를 해결하기 위해서 Docker-compose라는 것을 배워보며 문제를 해결해보도록 하겠습니다.

# 시리즈
[1. [Docker] 기본 개념 정리 및 자주 사용하는 Command](/infra/docker-basic-command)
[3. [Docker] Docker compose를 이용한 다중 컨테이너 구현](/infra/docker-compose)