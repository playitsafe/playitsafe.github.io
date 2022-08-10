---
title: Implement a countdown timer in React Hooks
date: 2022-04-02 04:10:12
categories: "Front-end"
tags:
- Hooks
- React
- Web
---
# Prefix
---
From time to time we will see timers in the world of frontend, of which the use cases could be a count-down function, or a disabled button which will be released active after a certain amount of time to force users to read through site T&C, or a button to send mobile code which is restricted to be clickable within a time interval after last click. Many front-end frameworks have already provided an out-of-box component of that such as [ProFormCaptcha](https://procomponents.ant.design/en-US/components/field-set#proformcaptcha) which to support common CAPTCHA functionality in the middle and backend.

However, we will run into some cases that we need to customize the function as we wish and the provided function from the framework may have limitations. In this case we will need to implement this function by ourself.


# Requirements Analysis
---
Ideally, we need to implement a button that behaves like this:
- First sms will be sent at the time the modal pops up, and the button start to count down util it reaches the time interval.
- After each count down, the button will be clickable and will start counting down after each click.

![captcha button gif](https://cdn.jsdelivr.net/gh/playitsafe/cdn/img/sigma-captcha.gif)

# Implementation in a React component
## Basic setup:
---
From the perspective of code, will need two states to manage this timer:
- A state to record the status of the button(disabled or active).
- A state to record time left after each seconds

So the basic code structure looks like this:
```tsx
import React, { useEffect, useState } from 'react';
import { Input, Button } from '@/my/components';

const TIME_INTERVAL = 59;

export default const TimerModal = () => {
	const [seconds, setSeconds] = useState<number>(TIME_INTERVAL);
	const [isCounting, setIsCounting] = useState<boolean>(false);

	const clickHandler = () => {
		sendCode();
		// reset timer
	}
	
	return (
		<Input />
		<Button onClick={clickHandler} disabled={isCounting} />
	);
}
```

## Add timer logic
From the requirement above we would know that the `TimerModal` will pop up and a first SMS has already been sent and the Timer button is disabled and start to count down. In this situation `useEffect` hook will be a perfect function to host the timer logic.

### `useEffect()` dependency list
To listen to the change of the timer button and ensure timer works as we expected, we need to put two of the states(`seconds` and `isCounting`) to the dependency list of `useEffect`. 

### Inside of the timer logic
We achieve the expected result by updating the `seconds`(minus 1 in each call) inside a `setInterval` function. The initial seconds right after the Timer component is mounted is 59s, in the meanwhile, `setInterval` function inside `useEffect` hook will update seconds by deducting 1, and this will also trigger useEffect hook again and clear previous setInterval function and start a new session of `setInterval()` to minus seconds by 1 util it counts down to 1 second, which means the setInterval function is cleared and the countdown session status is reset to inactive(to set `isCounting` to false and `seconds` set to initial time interval) and the button will be clickable.

After the button is clicked, the timer will be reset and a new cycle of the useEffect hook will start again.

```tsx
import React, { useEffect, useState } from 'react';
import { Input, Button } from '@/my/components';

const TIME_INTERVAL = 59;

export default const TimerModal = () => {
	const [seconds, setSeconds] = useState<number>(TIME_INTERVAL);
	const [isCounting, setIsCounting] = useState<boolean>(true);

	const resetTimer = () => {
		setSeconds(TIME_INTERVAL);
		setIsCounting(true);
	}

	useEffect((): () => void => {
		//setInterval Type would be number if uses window.setInterval()
		let interval: null | NodeJS.Timer = null;
		if (isCounting) {
			interval = setInterval(() => {
				setSeconds(seconds => {
					if (seconds > 1) {
						return seconds - 1;
					} else {
						interval && clearInterval(interval);
						setIsCounting(false);
						return TIME_INTERVAL;
					}
				} );
			}, 1000);
		} else {
			interval && clearInterval(interval);
		}
	}, [ isCounting, seconds ])

	const clickHandler = () => {
		sendCode();
		resetTimer();
	}

	return (
		<Input />
		<Button onClick={clickHandler} disabled={isCounting} />
	);
}
```

# Review
The implementation above seems to perfectly fit the use case, one problem still hides under the hood and will cause memory leakage. If you are on the halfway during countdown session and the Timer component is unmounted(since it's a modal and will be unmounted after it's closed), the `setInterval` function is still there in the background. Luckily, the `useEffect` hook provides a return callback function to allow us unsubscribe any side effects outside of the React lifecycle, which will be called before the component is unmounted.
In this case, we need to return a housekeeping function at the end of the `useEffect` callback function to clear the `setInterval` function. So the final code looks like this:

```tsx
import React, { useEffect, useState } from 'react';
import { Input, Button } from '@/my/components';

const TIME_INTERVAL = 59;

export default const TimerModal = () => {
	const [seconds, setSeconds] = useState<number>(TIME_INTERVAL);
	const [isCounting, setIsCounting] = useState<boolean>(false);

	const resetTimer = () => {
		setSeconds(TIME_INTERVAL);
		setIsCounting(true);
	}

	useEffect((): () => void => {
		//setInterval Type would be number if uses window.setInterval()
		let interval: null | NodeJS.Timer = null;
		if (isCounting) {
			interval = setInterval(() => {
				setSeconds(seconds => {
					if (seconds > 1) {
						return seconds - 1;
					} else {
						interval && clearInterval(interval);
						setIsCounting(false);
						return TIME_INTERVAL;
					}
				} );
			}, 1000);
		} else {
			interval && clearInterval(interval);
		}
		// unsubscription
		return () => interval && clearInterval(interval);
	}, [ isCounting, seconds ])

	const clickHandler = () => {
		sendCode();
		resetTimer();
	}

	return (
		<Input />
		<Button onClick={clickHandler} disabled={isCounting} />
	);
}
```

Another takeaway from this case is that when we update the state which depends on the previous state, we can use a callback function inside `useState()` hook.
the `setState()` function provided by useState hook accepts either a new state value or a callback function as an updater which returns a new state to set state based on the previous state.

```jsx
const App = () => {
  const [num, setNum] = useState(0); // same API as useState

  const handleClick = () => {
    setNum(prevState => prevState + 1);
  };
  return <button onClick={handleClick}>Increment</button>;
}
```

# Summary
This is a use case that I encountered during development and I think this would be a perfect example to learn and demonstrate the concept of `useEffect` and `useState` hooks in React.js. When functional component started to take over class-based component, I firstly use the `useEffect` hook the way that I use `componentDidMount()`, `componentDidUpdate()` and `componentDidUnmount()` function. After a while, I started to learn that they may share most of the use cases, but the conception is different and `useEffect` hooks tends to be a much cleaner way to implement and manage a single logic at one place instead of having to use multiple lifecycle functions to trigger/subscribe/unsubscribe side effects.

## References
React docs: https://reactjs.org/docs/hooks-effect.html