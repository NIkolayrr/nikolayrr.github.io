---
title: React
permalink: "/digital-garden/react/"
layout: topic
---

### Hooks

**useMemo()**: cache the result of a calculation between re-renders. useMemo should be used on complex calculations.

```js
const total = useMemo(() => numbers.reduce((acc, number) => acc + number, 0)),
[numbers] // anything you read from should go into the dependency array)
```

**useEffect()**: Timer component

```js
const Time = () => {
  const [time, setTime] = useState(0)
  useEffect(() => {
    setInterval(() => {
      setTime((t) => t + 1)
    }, 1000)

    return () => clearInterval(time)
  }, [])

  return <div>Time: {time}</div>
}
```
