# Math-game-app-13.08.25
import React, { useState, useEffect, useRef } from 'react';
import { AnimatePresence, motion } from 'framer-motion';

// A simple utility to play sounds using the browser's AudioContext
const playSound = (isCorrect) => {
  const audioContext = new (window.AudioContext || window.webkitAudioContext)();
  const oscillator = audioContext.createOscillator();
  const gainNode = audioContext.createGain();

  oscillator.connect(gainNode);
  gainNode.connect(audioContext.destination);

  if (isCorrect) {
    // Happy, ascending sound for a correct answer
    oscillator.type = 'sine';
    oscillator.frequency.setValueAtTime(600, audioContext.currentTime);
    oscillator.frequency.exponentialRampToValueAtTime(1000, audioContext.currentTime + 0.1);
    gainNode.gain.setValueAtTime(0.5, audioContext.currentTime);
    gainNode.gain.exponentialRampToValueAtTime(0.001, audioContext.currentTime + 0.3);
  } else {
    // A low, buzzing sound for an incorrect answer
    oscillator.type = 'sawtooth';
    oscillator.frequency.setValueAtTime(150, audioContext.currentTime);
    gainNode.gain.setValueAtTime(0.5, audioContext.currentTime);
    gainNode.gain.exponentialRampToValueAtTime(0.001, audioContext.currentTime + 0.5);
  }

  oscillator.start(audioContext.currentTime);
  oscillator.stop(audioContext.currentTime + 0.5);
};

// Main App component
function App() {
  const [score, setScore] = useState(0);
  const [question, setQuestion] = useState('');
  const [answer, setAnswer] = useState('');
  const [userInput, setUserInput] = useState('');
  const [feedback, setFeedback] = useState('');
  const [gameActive, setGameActive] = useState(false);
  const [questionCount, setQuestionCount] = useState(0);
  const [timeLeft, setTimeLeft] = useState(15);
  const maxQuestions = 10;
  const inputRef = useRef(null);
  const timerRef = useRef(null);

  // useEffect hook to handle the game timer
  useEffect(() => {
    if (gameActive) {
      // Set up the timer
      timerRef.current = setInterval(() => {
        setTimeLeft(prevTime => prevTime - 1);
      }, 1000);
    }

    return () => {
      // Clean up the timer when the component unmounts or game ends
      clearInterval(timerRef.current);
    };
  }, [gameActive]);

  // useEffect hook to handle time running out
  useEffect(() => {
    if (timeLeft === 0 && gameActive) {
      handleTimeout();
    }
  }, [timeLeft, gameActive]);

  // useEffect hook to focus on the input field when a new question appears
  useEffect(() => {
    if (gameActive && inputRef.current) {
      inputRef.current.focus();
    }
  }, [question, gameActive]);

  // Function to generate a random math problem for a 9-year-old
  const generateQuestion = () => {
    const num1 = Math.floor(Math.random() * 20) + 5; // Numbers from 5 to 24
    const num2 = Math.floor(Math.random() * 10) + 1; // Numbers from 1 to 10
    const operations = ['+', '-', '×', '÷'];
    const operation = operations[Math.floor(Math.random() * operations.length)];

    let newQuestion = '';
    let correctAnswer = 0;

    switch (operation) {
      case '+':
        newQuestion = `${num1} + ${num2}`;
        correctAnswer = num1 + num2;
        break;
      case '-':
        if (num1 < num2) {
          newQuestion = `${num2} - ${num1}`;
          correctAnswer = num2 - num1;
        } else {
          newQuestion = `${num1} - ${num2}`;
          correctAnswer = num1 - num2;
        }
        break;
      case '×':
        newQuestion = `${num1} × ${num2}`;
        correctAnswer = num1 * num2;
        break;
      case '÷':
        // Ensure the division always results in a whole number
        const product = num1 * num2;
        newQuestion = `${product} ÷ ${num1}`;
        correctAnswer = num2;
        break;
      default:
        break;
    }

    setQuestion(newQuestion);
    setAnswer(correctAnswer.toString());
    setUserInput('');
    setFeedback('');
    setQuestionCount(prevCount => prevCount + 1);
    setTimeLeft(15); // Reset the timer for the new question
  };

  // Function to start the game
  const startGame = () => {
    setScore(0);
    setQuestionCount(0);
    setGameActive(true);
    generateQuestion();
  };

  // Function to handle time running out
  const handleTimeout = () => {
    setFeedback(`Time's up! The answer was ${answer}. ⏱️`);
    playSound(false);
    
    setTimeout(() => {
      if (questionCount < maxQuestions) {
        generateQuestion();
      } else {
        setGameActive(false);
      }
    }, 1500);
  };

  // Function to handle user's answer submission
  const handleSubmit = () => {
    clearInterval(timerRef.current); // Stop the timer
    if (userInput.trim() === '') {
      setFeedback('Please enter an answer!');
      return;
    }

    const isCorrect = userInput === answer;
    playSound(isCorrect);

    if (isCorrect) {
      setFeedback('Correct! 🚀');
      setScore(prevScore => prevScore + 1);
    } else {
      setFeedback(`Incorrect. The answer was ${answer}. 😔`);
    }

    setTimeout(() => {
      if (questionCount < maxQuestions) {
        generateQuestion();
      } else {
        setGameActive(false);
      }
    }, 1500);
  };

  // Handle Enter key press for submission
  const handleKeyPress = (event) => {
    if (event.key === 'Enter') {
      handleSubmit();
    }
  };

  // UI for the progress bar
  const progressBar = (
    <div className="w-full bg-gray-200 rounded-full h-4 mb-4">
      <motion.div
        className="bg-green-500 h-4 rounded-full"
        initial={{ width: 0 }}
        animate={{ width: `${(questionCount / maxQuestions) * 100}%` }}
        transition={{ duration: 0.5 }}
      />
    </div>
  );

  // Render the game UI
  return (
    <div className="min-h-screen bg-gradient-to-br from-blue-700 to-indigo-900 flex items-center justify-center p-4 font-sans text-white">
      <div className="bg-white rounded-[40px] shadow-2xl p-8 max-w-lg w-full text-center transition-all duration-500 transform hover:scale-[1.02] text-gray-800">
        <motion.h1
          className="text-5xl font-extrabold text-blue-800 mb-6"
          initial={{ y: -50, opacity: 0 }}
          animate={{ y: 0, opacity: 1 }}
          transition={{ type: "spring", stiffness: 100 }}
        >
          Space Math Mission! 🌌
        </motion.h1>

        {!gameActive ? (
          // Start and End screens
          <motion.div
            className="space-y-8"
            initial={{ scale: 0.9, opacity: 0 }}
            animate={{ scale: 1, opacity: 1 }}
            transition={{ type: "spring", stiffness: 200, delay: 0.2 }}
          >
            <p className="text-2xl text-gray-600 font-medium">
              Ready for a mission to test your skills?
            </p>
            <button
              onClick={startGame}
              className="w-full bg-green-500 hover:bg-green-600 text-white font-bold py-5 px-6 rounded-full shadow-lg transform transition-all duration-300 hover:scale-110 focus:outline-none focus:ring-4 focus:ring-green-300 active:bg-green-700 text-2xl"
            >
              Start Mission!
            </button>
            {questionCount === maxQuestions && (
              <motion.div
                className="mt-6"
                initial={{ opacity: 0 }}
                animate={{ opacity: 1 }}
                transition={{ delay: 1 }}
              >
                <p className="text-3xl font-extrabold text-purple-700">
                  Mission Complete!
                </p>
                <p className="text-4xl font-extrabold text-indigo-600 mt-2">
                  Final Score: {score} / {maxQuestions}
                </p>
              </motion.div>
            )}
          </motion.div>
        ) : (
          // Game in progress
          <motion.div
            className="space-y-8"
            initial={{ scale: 0.9, opacity: 0 }}
            animate={{ scale: 1, opacity: 1 }}
            transition={{ type: "spring", stiffness: 200, delay: 0.2 }}
          >
            <div className="flex justify-between items-center mb-6">
              <p className="text-xl font-semibold text-gray-600">
                Score: <span className="text-green-500 text-3xl font-extrabold">{score}</span>
              </p>
              <p className="text-xl font-semibold text-gray-600">
                Time: <span className="text-red-500 text-3xl font-extrabold">{timeLeft}s</span>
              </p>
            </div>
            {progressBar}
            
            <motion.div
              key={question}
              initial={{ x: -100, opacity: 0 }}
              animate={{ x: 0, opacity: 1 }}
              transition={{ type: "spring", stiffness: 100, damping: 10 }}
            >
              <p className="text-7xl font-bold text-purple-700 mb-8">
                {question} = ?
              </p>
            </motion.div>

            <input
              ref={inputRef}
              type="number"
              value={userInput}
              onChange={(e) => setUserInput(e.target.value)}
              onKeyPress={handleKeyPress}
              placeholder="Your Answer"
              className="w-full p-5 text-4xl text-center border-4 border-blue-300 rounded-2xl focus:outline-none focus:border-blue-500 transition-all duration-300 shadow-lg"
            />

            <AnimatePresence>
              {feedback && (
                <motion.p
                  key="feedback"
                  initial={{ y: 20, opacity: 0 }}
                  animate={{ y: 0, opacity: 1 }}
                  exit={{ y: -20, opacity: 0 }}
                  transition={{ duration: 0.5 }}
                  className="text-3xl font-extrabold mt-4 h-8"
                  style={{ color: feedback.includes('Correct') ? '#22C55E' : '#EF4444' }}
                >
                  {feedback}
                </motion.p>
              )}
            </AnimatePresence>

            <button
              onClick={handleSubmit}
              className="w-full bg-blue-500 hover:bg-blue-600 text-white font-bold py-5 px-6 rounded-full shadow-lg transform transition-all duration-300 hover:scale-110 focus:outline-none focus:ring-4 focus:ring-blue-300 active:bg-blue-700 text-2xl"
            >
              Submit Answer
            </button>
          </motion.div>
        )}
      </div>
    </div>
  );
}

export default App;
