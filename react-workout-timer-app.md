Cardio HIIT Timer App in React-Native
26-05-2022


[High Intensity Interval Training](https://www.healthline.com/nutrition/benefits-of-hiit) has been catching on lately (or maybe I have finally caught up with it). It seems like an awesome way to workout in the confines of my home. They are basically short bursts of intense exercise followed by a rest period. The workouts require a way to keep track of the time and the reps. Unfortunately there seems to be no app (web or otherwise) catering to this specific need. So I thought, what the hell, I’ll just build one on my own.

### The Tech

Nothing fancy, just some javascript. I used [React-Native](https://reactnative.dev/) for the app. The app has been tested only on Android. Testing on Apple products is in the works. The main idea is to take the number of repetitions, repetition time and rest time as inputs. Then have a timer run down from rep-time to 0, rest-time to 0 and repeat for number of reps. Beep whenever the app state changes from rep. to rest. Simple enough. The timer can be handled using javascipt’s **setInterval** function.

![fitness app](https://upload.wikimedia.org/wikipedia/commons/thumb/3/3c/Calorie_counter_-_Day_summary_%28dark_theme_-_English%29.png/235px-Calorie_counter_-_Day_summary_%28dark_theme_-_English%29.png)

I ran into issues as soon as I started writing the logic. It turns out, React-Native does not handle the setInterval function well. Reading and updating state variables from inside a setInterval block is not a trivial matter. After hours of googling & searching on [StackOverflow](https://stackoverflow.com/questions/26348557/issue-accessing-state-inside-setinterval-in-react-js), I figured out a way to use react-native’s [hooks](https://reactjs.org/docs/hooks-effect.html) functionality, specifically **useEffects** to get it to work. Another issue I ran into, was the setInterval delay timer. Since react-native runs only on one thread, it is [not guaranteed](https://andrewduthie.com/2013/12/31/creating-a-self-correcting-alternative-to-javascripts-setinterval/) that setInterval will actally run exactly after every _delay_ milliseconds. This is quite a fundamental issue and the only way to alleviate this is to have self-correcting logic inside the setInterval block using _performance.now_. I borrowed the coed for the self-adjusting _setInterval_ function from [andrewduthie.com](https://andrewduthie.com/2013/12/31/creating-a-self-correcting-alternative-to-javascripts-setinterval/). I used johnsonsu’s [react-native-sound-player](https://github.com/johnsonsu/react-native-sound-player#readme) for the beep functionality . A functionality, I added in later is the ability to keep the screen awake while the app is being used. For this, I used sayem314’s [react-native-keep-awake](https://www.npmjs.com/package/@sayem314/react-native-keep-awake) package.

```
useEffect(() => {
    let timer = null;

    if(!isPaused){
      timer = setInterval(() => {
        setCurrTime(currTime => currTime + 1);
        if((currTime >= repTime) && isResting==false ){
          //TODO - beep
          setCurrTime(0);
          setResting(true);
        }

        if((currTime >= restTime) && isResting==true ){
          //TODO - beep
          setCurrTime(0);
          setResting(false);
          setCurrRep(currentRep => currentRep + 1);
        }
      },1000)
    } else if (isPaused) {
      clearInterval(timer);
    }
    return () => clearInterval(timer);
  },[isPaused,currTime,repTime,isResting,restTime,currentRep,sets]);
```


The next part that bothered me was the UI and the color scheme. Aesthetic Design does not come to me easily. But I found an awesome resource for figuring out the color schemes - [Coolors](https://coolors.co/). It generates random color palettes that work well together.

This app only took about 2 days for me to create from scratch. I’ll try to make the UI more appealling. Get some [material design](https://material-ui.com/) components, make the timer look like an actual timer instead of just text. Maybe display some ads and earn some money (why not!!).