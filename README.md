# farm-assignment
import React, { useRef } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { PanGestureHandler, State } from 'react-native-gesture-handler';
import { spring, useSpring, animated } from 'react-spring/native';

const AnimatedView = animated(View);

const BottomSheet = ({ snapPoints }) => {
  const translateY = useRef(new animated.Value(snapPoints[0])).current;

  const onGestureEvent = ({ nativeEvent }) => {
    const { translationY } = nativeEvent;
    translateY.setValue(translationY);
  };

  const onHandlerStateChange = ({ nativeEvent }) => {
    if (nativeEvent.oldState === State.ACTIVE) {
      const { translationY, velocityY } = nativeEvent;

      // Calculate the index of the nearest snap point based on velocity
      const index = snapPoints.reduce(
        (prevIndex, snapPoint, currentIndex) => {
          const distanceToSnapPoint =
            Math.abs(translationY - snapPoint);
          const prevDistance =
            Math.abs(translationY - snapPoints[prevIndex]);
          return distanceToSnapPoint < prevDistance
            ? currentIndex
            : prevIndex;
        },
        0
      );

      // Spring to the nearest snap point
      spring(translateY, {
        toValue: snapPoints[index],
        velocity: velocityY,
        useNativeDriver: false,
      }).start();
    }
  };

  const animatedStyle = useSpring({
    translateY: translateY,
    useNativeDriver: false,
  });

  return (
    <PanGestureHandler
      onGestureEvent={onGestureEvent}
      onHandlerStateChange={onHandlerStateChange}
    >
      <AnimatedView
        style={[styles.container, animatedStyle]}
        pointerEvents="box-none"
      >
        {/* Content of your bottom sheet */}
        <View style={styles.content}>
          <Text>Bottom Sheet Content</Text>
        </View>
      </AnimatedView>
    </PanGestureHandler>
  );
};

const styles = StyleSheet.create({
  container: {
    position: 'absolute',
    bottom: 0,
    left: 0,
    right: 0,
    backgroundColor: 'white',
    borderTopLeftRadius: 16,
    borderTopRightRadius: 16,
    padding: 16,
  },
  content: {
    alignItems: 'center',
    justifyContent: 'center',
  },
});

export default BottomSheet;


Using bottom sheet component:
-----------------------------
import React from 'react';
import { View, StyleSheet } from 'react-native';
import BottomSheet from './BottomSheet';

const App = () => {
  // Define your snap points (e.g., 0, 300, 600)
  const snapPoints = [0, 300, 600];

  return (
    <View style={styles.container}>
      {/* Your main content */}
      <View style={styles.content}>
        <Text>Main Content</Text>
      </View>

      {/* Bottom Sheet */}
      <BottomSheet snapPoints={snapPoints} />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  content: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default App;

