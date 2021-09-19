---
templateKey: post
title: useTwoKeyboardKeysEvent
date: "2020-19-09"
gist: https://gist.github.com/Miloshinjo/8d819a2b0059afe428088ccbdb8319bb
sandbox: https://codesandbox.io/s/black-moon-xhhpp
isMultilingual: false
---

What this hook does is that allows you to call event handlers when two keyboard keys are pressed. You can invoke the event by holding the first key down and then pressing another. Hook is specialized for NextJS, but you can adjust it to fit your project needs.

```typescript
import { useCallback, useState } from 'react';

// Usage
import { useCallback } from "react";
import { useTwoKeyboardKeysEvent } from "../hooks/useTwoKeyboardKeysEvent";

const IndexPage = () => {
  const eventHandler = useCallback(() => {
    alert("Keys pressed event fired");
  }, []);

  useTwoKeyboardKeysEvent({
    keys: ["d", "j"],
    eventHandler
  });

  return (
    <div>
      Hello useTwoKeyboardKeysEvent. Hold D and press J to see if it works.
    </div>
  );
};

export default IndexPage;

// Hook
import { useRouter } from "next/router";
import { useEffect, useState } from "react";

type UseTwoKeyboardKeysEventParams = {
  keys: [string, string];
  eventHandler: (event: KeyboardEvent) => void;
};

/**
 * React hook that dispatches an event after holding down one key and pressing
 * another key. eg. (⌘ + j). Specialized for nextjs.
 *
 * @param keys          A tuple of 2 keys taken from event.key.
 * @param eventHandler  Callback we want to invoke. Make sure to pass it via useCallback.
 */
export const useTwoKeyboardKeysEvent = ({
  keys,
  eventHandler
}: UseTwoKeyboardKeysEventParams) => {
  const [keyOne, keyTwo] = keys;
  const router = useRouter();
  const [isKeyOneHeld, setKeyOneHeld] = useState(false);

  useEffect(() => {
    /**
     * KeyDown handler
     */
    const handleKeyDown = (event: KeyboardEvent) => {
      if (event.key === keyOne) {
        setKeyOneHeld(true);
      }

      // Only call callback when first key is held
      if (event.key === keyTwo && isKeyOneHeld === true) {
        eventHandler(event);
      }
    };

    // If you want it to not be global (attached to the document object), you can change it to be attached to the ref, just pass it as an argument.
    document.addEventListener("keydown", handleKeyDown);

    return () => {
      document.removeEventListener("keydown", handleKeyDown);
    };
  }, [isKeyOneHeld, eventHandler, keyOne, keyTwo]);

  useEffect(() => {
    /**
     * KeyUp handler
     */
    const handleKeyUp = (event: KeyboardEvent) => {
      if (event.key === keyOne) {
        setKeyOneHeld(false);
      }
    };

    // If you want it to not be global (attached to the document object), you can change it to be attached to the ref, just pass it as an argument.
    document.addEventListener("keyup", handleKeyUp);

    return () => {
      document.removeEventListener("keyup", handleKeyUp);
    };
  }, [isKeyOneHeld, keyOne]);

  useEffect(() => {
    /**
     * In order to reset it in case of changing pages while holding keyOne. This is
     * NextJS specialized, if not on NextJS feel free to handle this behaviour somehow or not.
     */
    const handleRouteChange = () => {
      setKeyOneHeld(false);
    };

    router.events.on("routeChangeComplete", handleRouteChange);

    return () => {
      router.events.off("routeChangeComplete", handleRouteChange);
    };
    /**
     * This only needs to run on mount
     */
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);
};
```
