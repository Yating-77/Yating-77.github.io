<template>
  <div class="fullscreen-container">
    <div v-if="!showResults" class="row g-0 h-100">
      <!-- Left fixed image area -->
      <div class="col-md-5 d-flex align-items-center justify-content-center sidebar-container">
        <div class="position-sticky sidebar-content">
          <h1 class="text-center mb-3 page-title">Supporting Autism Families</h1>
          <p class="text-center mb-4 page-subtitle">
            Personalized recommendations based on your child's daily behaviors
          </p>
          <div class="main-image-container">
            <img 
              :src="'/scenario-' + currentScenario + '.jpg'" 
              alt="Scenario Illustration" 
              class="img-fluid main-image"
              onerror="this.src='/sleep_scenario_example.png'"
            />
          </div>
          
          <!-- Progress indicator -->
          <div class="progress-container mt-4">
            <div class="d-flex justify-content-between mb-2">
              <span class="progress-text">Scenario {{ currentScenario + 1 }}/{{ scenarios.length }}</span>
              <span class="progress-percentage">{{ Math.round(((currentScenario + 1) / scenarios.length) * 100) }}% Complete</span>
            </div>
            <div class="progress">
              <div class="progress-bar" 
                   :style="{ width: ((currentScenario + 1) / scenarios.length * 100) + '%' }">
              </div>
            </div>
            <div class="scenario-dots mt-3 d-flex justify-content-between">
              <div v-for="(scenario, index) in scenarios" :key="index" 
                   class="scenario-dot" 
                   :class="{ 'active': index <= currentScenario, 'current': index === currentScenario }"
                   @click="navigateToScenario(index)">
              </div>
            </div>
          </div>
        </div>
      </div>
      
      <!-- Right question and options area -->
      <div class="col-md-7 d-flex align-items-center">
        <div class="card w-100 border-0 shadow-sm content-card">
          <div v-if="!currentSelectedOption" class="card-body p-md-5 p-4">
            <div class="scenario-title mb-2">
              {{ scenarios[currentScenario].title }}
            </div>
            <h3 class="mb-4 scenario-question">
              {{ scenarios[currentScenario].question }}
            </h3>
            
            <div class="options-container">
              <div 
                v-for="(opt, index) in scenarios[currentScenario].options" 
                :key="index"
                class="option-card my-3 p-3" 
                :class="{'selected-option': selected === index}"
                @click="selected = index"
              >
                <input class="form-check-input visually-hidden" type="radio" :id="'opt' + index" :value="index" v-model="selected">
                <label class="form-check-label d-block" :for="'opt' + index">
                  <div class="d-flex align-items-center">
                    <div class="option-indicator me-3" :class="{'selected-indicator': selected === index}"></div>
                    <div class="option-text">{{ opt.text }}</div>
                  </div>
                </label>
              </div>
              
              <button 
                class="btn mt-4 w-100 py-3 action-button" 
                @click="showRecommendation" 
                :disabled="selected === null"
              >
                View Recommendations
              </button>
            </div>
          </div>
          
          <div v-else class="recommendation-container card-body p-md-5 p-4">
            <div class="scenario-title mb-2">
              {{ scenarios[currentScenario].title }}
            </div>
            <h3 class="mb-4 recommendations-heading">Professional Recommendations</h3>
            
            <div class="recommendation-content p-4 mb-4">
              <h4 class="mb-3 recommendation-subtitle">Recommendations for "{{ currentSelectedOption.text }}"</h4>
              
              <div class="recommendation-cards">
                <div v-for="(rec, i) in formatRecommendations(currentSelectedOption.recommendations)" :key="i" 
                     class="recommendation-card mb-4">
                  <div class="card-header d-flex align-items-center p-3">
                    <div class="rec-icon me-3">
                      <i class="fas fa-check-circle"></i>
                    </div>
                    <h5 class="m-0 recommendation-title">{{ rec.title }}</h5>
                  </div>
                  
                  <div class="card-body p-3">
                    <div class="content">
                      {{ rec.content }}
                    </div>
                    
                    <div v-if="rec.example" class="example mt-3 p-2">
                      <div class="example-title">Practical Application:</div>
                      <div class="example-content">{{ rec.example }}</div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
            
            <div class="d-flex justify-content-between mt-4">
              <button 
                class="btn back-button" 
                @click="resetCurrentScenario"
              >
                Choose Again
              </button>
              
              <button 
                v-if="currentScenario < scenarios.length - 1"
                class="btn next-button" 
                @click="navigateToNextScenario"
              >
                Next Scenario
              </button>
              
              <button 
                v-else
                class="btn finish-button" 
                @click="completeScenarios"
              >
                Complete
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- Results Page -->
    <div v-if="showResults" class="results-container p-4">
      <div class="results-card card border-0 shadow-sm p-4">
        <div class="d-flex justify-content-between align-items-center mb-4">
          <h1 class="results-title">Your Personalized Recommendations</h1>
          <button class="btn download-button" @click="downloadPDF">
            <i class="fas fa-download me-2"></i> Download PDF
          </button>
        </div>
        
        <div class="results-intro mb-4 p-3">
          <h4 class="results-intro-title">Based on your selections, we've prepared the following recommendations for your child</h4>
          <p>These recommendations are tailored to the behavioral patterns you've described and may help your child. Remember that each child is unique, and these suggestions may need to be adjusted to fit your child's specific needs.</p>
        </div>
        
        <div id="results-content">
          <div v-for="(selection, index) in userSelections" :key="index" class="selection-result mb-5">
            <div class="scenario-header d-flex align-items-center mb-3">
              <div class="scenario-number">
                {{ index + 1 }}
              </div>
              <div>
                <h3 class="scenario-result-title">{{ selection.scenarioTitle }}</h3>
                <p class="mb-0 scenario-result-question">{{ selection.scenarioQuestion }}</p>
              </div>
            </div>
            
            <div class="selected-option-box p-3 mb-3">
              <h5 class="selected-option-title">Your Selection: {{ selection.optionText }}</h5>
            </div>
            
            <div class="recommendations-summary">
              <h5 class="recommendations-summary-title">Professional Recommendations:</h5>
              <div class="recommendation-list">
                <div v-for="(rec, recIndex) in formatRecommendations(selection.recommendations)" :key="recIndex" 
                    class="recommendation-item mb-4 p-3">
                  <h6 class="recommendation-item-title">{{ rec.title }}</h6>
                  <p class="mb-2">{{ rec.content }}</p>
                  <div class="example p-2 mt-2">
                    <div class="example-title">Practical Application:</div>
                    <div class="example-content">{{ rec.example }}</div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
        
        <div class="d-flex justify-content-between mt-4">
          <button class="btn back-button" @click="backToScenarios">
            Back
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      selected: null,
      currentSelectedOption: null,
      currentScenario: 0,
      showResults: false,
      userSelections: [],
      scenarios: [
        {
          title: 'Sleep Issues',
          question: 'Which of the following best describes your child\'s behavior at bedtime?',
          options: [
            { 
              text: 'Difficulty falling asleep (unable to quickly settle down for sleep)',
              recommendations: [
                {
                  title: 'Establish a Bedtime Routine',
                  content: 'Create a predictable bedtime process that helps your child gradually relax and prepare for sleep.',
                  example: 'For example: Bath → Pajamas → Brush teeth → Read a quiet story → Goodnight kiss → Lights out. Keep the same sequence and timing every night.'
                },
                {
                  title: 'Create a Comfortable Sleep Environment',
                  content: 'Ensure your child\'s sleep environment is comfortable and safe, which helps them fall asleep faster.',
                  example: 'Choose appropriate mattress and pillows, keep room temperature between 65-72°F (18-22°C), use a soft night light if your child is afraid of the dark.'
                },
                {
                  title: 'Limit Screen Time',
                  content: 'Avoid electronic devices at least one hour before bedtime, as blue light interferes with melatonin production.',
                  example: 'After dinner, replace TV or tablet time with quiet board games, puzzles, or drawing activities.'
                },
                {
                  title: 'Practice Relaxation Activities',
                  content: 'Engage in quiet, calming activities before bed to help your child relax physically and mentally.',
                  example: 'Listen to gentle music, practice simple breathing exercises, read or tell soothing bedtime stories.'
                },
                {
                  title: 'Increase Daytime Physical Activity',
                  content: 'Ensure your child gets plenty of outdoor activities and exercise during the day, making it easier to fall asleep after expending energy.',
                  example: 'Schedule at least 1-2 hours of outdoor play daily, such as running, biking, ball games, or playground time, but avoid vigorous exercise within 3 hours of bedtime.'
                }
              ]
            },
            { 
              text: 'Wakes up frequently during the night and has trouble falling back asleep (poor sleep continuity)',
              recommendations: [
                {
                  title: 'Maintain Consistent Sleep Environment',
                  content: 'Have your child fall asleep in a familiar environment where they will feel secure if they wake up during the night.',
                  example: 'Help your child fall asleep in their own bed, rather than on your bed or the couch. Avoid using TV or bright lights as sleep aids.'
                },
                {
                  title: 'Calmly Respond to Night Wakings',
                  content: 'When your child wakes during the night, keep interactions minimal and help them return to sleep.',
                  example: 'Quietly say "It\'s time to sleep now," gently guide your child to lie down, and avoid turning on lights, playing, or extended conversations.'
                },
                {
                  title: 'Use Comfort Items',
                  content: 'Provide items that give your child a sense of security to help them self-soothe.',
                  example: 'Use a special stuffed animal, soft blanket, or favorite pillow as a "sleep buddy" placed within easy reach.'
                },
                {
                  title: 'Create Consistent Sensory Environment',
                  content: 'Maintain consistent sound, light, and temperature in the sleep environment to reduce stimuli that might wake your child.',
                  example: 'Use a white noise machine or fan to provide consistent background sound, use blackout curtains to keep the room dark, and set the thermostat to maintain a stable temperature.'
                },
                {
                  title: 'Build Independent Sleep Skills',
                  content: 'Teach your child self-soothing skills for both falling asleep and night wakings.',
                  example: 'Gradually reduce parental intervention, from lying next to your child to just sitting in the room, and finally to briefly checking in. Encourage your child to use simple relaxation techniques like deep breathing.'
                }
              ]
            },
            { 
              text: 'Shows significant anxiety or crying when bedtime routine changes (overly rigid bedtime routine)',
              recommendations: [
                {
                  title: 'Announce Changes in Advance',
                  content: 'Before changing bedtime routines, inform your child ahead of time in a way they can understand.',
                  example: 'Use simple visual schedules or picture cards showing what will be different that night. For example: "Tonight daddy will read you a story instead of mommy."'
                },
                {
                  title: 'Introduce Changes Gradually',
                  content: 'Help your child adapt to adjustments in the bedtime routine through small steps.',
                  example: 'Start by changing less important items, like switching to a different bedtime story book; after success, try changing bath time or the location for story reading.'
                },
                {
                  title: 'Keep Core Elements Stable',
                  content: 'When changes are necessary, try to maintain key aspects of the routine unchanged to provide security.',
                  example: 'If you can\'t follow the usual bath routine, still maintain the same pajamas, teeth brushing, and story sequence, using the same language prompts: "Now it\'s time to get ready for bed."'
                },
                {
                  title: 'Create Flexibility Games',
                  content: 'Play "change games" during daytime to help your child practice adapting to small changes.',
                  example: 'During playtime, role-play different bedtime scenarios: "Today we\'re pretending we\'re traveling, and bedtime might be a bit different." Or slightly mix up the order of daily activities to help your child learn to adapt.'
                },
                {
                  title: 'Positively Reinforce Adaptive Behavior',
                  content: 'Praise and encourage your child when they successfully handle changes.',
                  example: 'Use a sticker reward system or special praise: "You were so brave tonight, even though bedtime was a little different, you stayed calm!" The next day, narrate how the child successfully adapted to the change.'
                }
              ]
            },
            { 
              text: 'Has difficulty falling asleep when there are slight changes in the sleep environment (sensitive to sleep environment)',
              recommendations: [
                {
                  title: 'Create a Portable Sleep Kit',
                  content: 'Prepare a set of familiar sleep items that can be used in different environments.',
                  example: 'Prepare a "sleep travel pack" containing your child\'s familiar pillowcase, small blanket, favorite stuffed toy, and possibly regular bedtime reading materials.'
                },
                {
                  title: 'Use Sensory Aids',
                  content: 'Utilize tools to block or regulate environmental distractions.',
                  example: 'Use a white noise machine to mask unfamiliar sounds, comfortable earplugs to reduce noise disruptions, sleep masks to adjust to light changes, or small items with familiar scents for comfort.'
                },
                {
                  title: 'Preview and Familiarize New Environments',
                  content: 'Before sleeping in a new place, help your child become familiar with the environment.',
                  example: 'Visit the new sleeping area in advance, allow your child to explore and choose where they\'ll sleep, do some enjoyable daytime activities in the new environment to build positive associations.'
                },
                {
                  title: 'Maintain Core Bedtime Routine',
                  content: 'Even in different environments, stick to key bedtime routine elements.',
                  example: 'No matter where you are, maintain the same bedtime three-step process, such as: brushing teeth, reading a story, and singing a specific lullaby, providing continuity and security through process.'
                },
                {
                  title: 'Create "Sleep Space" Boundaries',
                  content: 'Use visual or tactile cues to define the sleeping area.',
                  example: 'Use the same sheets or blanket to "mark" the sleep space, even when traveling; or use a foldable tent-style bed cover to create enclosure, reducing external environment interference.'
                }
              ]
            }
          ]
        },
        {
          title: 'Diet and Nutrition Issues',
          question: 'During mealtimes, how does your child typically behave?',
          options: [
            { 
              text: 'Refuses to try new foods, only eats a few specific foods (narrow food acceptance)',
              recommendations: [
                {
                  title: 'Introduce New Foods Gradually',
                  content: 'Use a "food bridge" strategy by placing new foods alongside familiar foods that your child enjoys.',
                  example: 'Place a small piece of carrot next to foods with similar flavors that they already like. Encourage your child to first observe, smell, or even touch the new food without forcing them to eat it.'
                },
                {
                  title: 'Repeated Exposure',
                  content: 'You may need to present foods 5-20 times before acceptance—research shows children with autism may need 10+ exposures before trying new foods.',
                  example: 'Be patient and continue offering the same new food at different meals. Acceptance typically increases gradually with continued neutral exposure.'
                },
                {
                  title: 'Provide Safety with Familiar Foods',
                  content: 'When offering new foods, make sure there are always some familiar foods on the plate.',
                  example: 'Ensure that at least half the plate contains foods you know your child will eat, so they won\'t refuse the entire meal when presented with new options.'
                }
              ]
            },
            { 
              text: 'Extremely sensitive to food texture or taste, leading to limited diet (sensory food aversions)',
              recommendations: [
                {
                  title: 'Work with Texture Preferences',
                  content: 'For children sensitive to texture, first accept the textures they can tolerate while ensuring nutritional intake. Then gradually introduce subtle texture changes.',
                  example: 'If they only eat pureed foods, try introducing minimal soft particles and gradually increase the amount. If they only eat crunchy foods, try offerings that start crunchy but become slightly softer.'
                },
                {
                  title: 'Sensory Food Play',
                  content: 'Outside of mealtimes, engage in play activities with different food textures without pressure to eat.',
                  example: 'Create games touching, squishing or sorting foods with different textures. This helps desensitize children to textures they find challenging in a low-pressure environment.'
                }
              ]
            },
            { 
              text: 'Irregular eating times or difficulty sitting calmly at the table (poor mealtime structure)',
              recommendations: [
                {
                  title: 'Establish Regular Meal Schedule',
                  content: 'Create and strictly follow a daily schedule of three meals and two snacks for your child.',
                  example: 'Children with autism typically prefer predictable routines. Fixed mealtimes help establish biological rhythms and hunger-satiety patterns. For example, breakfast at 7 AM, lunch at noon, dinner at 6 PM, with scheduled snacks in between.'
                },
                {
                  title: 'Use Visual Supports',
                  content: 'Implement visual schedules or timers to help your child understand when it\'s time to eat and when mealtime ends.',
                  example: 'Before meals, show your child a picture schedule saying, "Look, the clock is pointing to the apple, it\'s time for lunch." During meals, use a simple hourglass timer so they can see how long the meal will last.'
                }
              ]
            }
          ]
        },
        {
          title: 'Social Interaction Issues',
          question: 'When your child is with other children their age, what typically happens?',
          options: [
            { 
              text: 'Plays completely alone, rarely initiates contact with others (low social initiation)',
              recommendations: [
                {
                  title: 'Accept and Guide Gradually',
                  content: 'Understand that some children with autism prefer to play alone, which is part of their social development stage.',
                  example: 'When taking your child to places with other children, let them play by themselves at first without forcing them to join in. Even "parallel play" (playing alongside peers) is a first step toward social interaction.'
                },
                {
                  title: 'Create Parallel Play Opportunities',
                  content: 'Invite one or two familiar children to your home and prepare identical sets of toys for each child.',
                  example: 'Set up two puzzle sets or two boxes of blocks, allowing your child and their peer to play separately but in the same area. They may not interact much, but will become aware of each other\'s presence, helping your child gradually become comfortable with having companions nearby.'
                },
                {
                  title: 'Use Common Interests for Friendship',
                  content: 'Use your child\'s interests as a bridge to introduce peers and encourage social interaction.',
                  example: 'If your child loves train models, take them to a children\'s train toy group activity where other children share this interest. The common interest will make them more willing to stay among peers because everyone is engaged with something they enjoy. Similarly, invite a child who likes trains to play "train games" at home, each pushing their own trains but on the same track.'
                },
                {
                  title: 'Gradually Increase Interaction',
                  content: 'Once your child is comfortable having peers nearby, slowly introduce simple interactions that don\'t require complex social skills.',
                  example: 'Play a ball-passing game: sit in a small circle with your child and another child, gently pushing the ball to the next person. This non-verbal interaction allows your child to experience the joy of playing with others without pressure. When they participate willingly, smile and encourage them by saying, "It\'s so fun to play together!"'
                }
              ]
            },
            { 
              text: 'Shows interest in other children but doesn\'t know how to interact (lacks interaction skills)',
              recommendations: [
                {
                  title: 'Directly Teach Social Skills',
                  content: 'Use role-playing and other methods to demonstrate and practice basic interaction skills.',
                  example: 'Play pretend games with your child: you be the "friend" and your child is "themselves," practicing greetings and responses—"Hello, my name is ___, would you like to play with me?" Through this simulation, they can try and learn in a safe environment before applying these skills with real peers.'
                },
                {
                  title: 'Use Visual Aids and Social Stories',
                  content: 'Create picture cards or short social scripts to help your child understand how to initiate interactions.',
                  example: 'Make a card showing two children exchanging toys with text saying "Can I play with your toy? You can play with mine." Practice this scenario with your child, teaching them to say the phrases on the card. Or use short social story books describing scenarios like "Tom wanted to play with friends but didn\'t know what to say. He watched what they were playing, then smiled and asked \'Can I join?\' They had fun playing together."'
                },
                {
                  title: 'Arrange One-on-One Structured Play',
                  content: 'Purposefully create small-scale interaction opportunities with clear structure and guidance.',
                  example: 'Invite a child of similar age with a gentle temperament to your home to play for half an hour. Prepare structured activities like completing a simple puzzle or building small block towers together. During the activity, guide them to take turns: "Your turn to place a piece, then your friend\'s turn." This teaches waiting and cooperation skills.'
                },
                {
                  title: 'Teach Help-Seeking Methods',
                  content: 'When your child wants to join others but doesn\'t know how to speak up, provide them with a simple way to ask for help.',
                  example: 'Teach your child to say "Mom, can you help me?" when they want to join a group activity, allowing you to facilitate the introduction. Or have them carry a card that says "Can I play too?" to give to a teacher or another parent when they want to participate. This reduces social pressure while they learn to initiate on their own.'
                }
              ]
            },
            { 
              text: 'Frequently has conflicts with peers or struggles to cooperate (difficulty with social-emotional management)',
              recommendations: [
                {
                  title: 'Teach "Calm Break" Strategy',
                  content: 'Show your child how to use a pause strategy instead of engaging in direct conflict when feeling upset during play.',
                  example: 'Agree on a hand signal (like raising a palm) that means "I need to calm down." When they feel like losing control during peer interaction, they can use this gesture or say "stop," then step aside to take a few deep breaths or get a hug from you before returning. This prevents conflict escalation while teaching appropriate emotion regulation.'
                },
                {
                  title: 'Practice Conflict Resolution',
                  content: 'Use family time to role-play common conflicts and help your child practice appropriate responses.',
                  example: 'Simulate scenarios like "a friend took my toy" or "I\'m upset because I lost the game," then guide your child to suggest solutions: "I could tell my friend \'I\'m not finished yet, please wait\'" or "Take a deep breath and say \'let\'s play again\'". You can also create a "problem-solving jar" with various social conflict situations written on paper strips, regularly practicing solutions through role-play.'
                },
                {
                  title: 'Establish Clear Rules and Supervision',
                  content: 'Set up clear expectations for playtime and provide appropriate oversight to prevent conflicts.',
                  example: 'Establish and display rules for play (taking turns, no grabbing, using words to express displeasure) as visual reminders. Stay nearby during peer interactions to intervene gently when you notice early signs of frustration. When your child successfully shares, takes turns, or compromises, immediately praise the specific behavior: "You let your friend go first, which made playing together much more fun!"'
                }
              ]
            },
            { 
              text: 'Approaches others but in inappropriate ways (poor social approach skills)',
              recommendations: [
                {
                  title: 'Teach Appropriate Distance and Greeting',
                  content: 'Help your child understand personal space boundaries and appropriate ways to greet others.',
                  example: 'Practice the "arm\'s length rule" for personal space. Role-play different greetings for different people (friends, teachers, strangers). Use social stories with pictures to illustrate these concepts.'
                },
                {
                  title: 'Develop Turn-Taking Skills',
                  content: 'Use structured activities to teach the back-and-forth nature of social interaction.',
                  example: 'Start with simple turn-taking games like rolling a ball back and forth, then progress to board games and conversation exchanges. Use visual supports like a "talking stick" that gets passed to indicate whose turn it is to speak.'
                },
                {
                  title: 'Provide Concrete Feedback',
                  content: 'Give immediate, specific feedback about social interactions in a supportive way.',
                  example: 'When your child interrupts, instead of saying "That\'s rude," try "When you talk while Sarah is speaking, she can\'t finish her thought. Let\'s wait until she\'s done, then you can share your idea."'
                }
              ]
            },
            { 
              text: 'Focuses only on preferred activities or interests, ignoring other children\'s participation or emotions (overly focused on personal interests)',
              recommendations: [
                {
                  title: 'Socialize Personal Interests',
                  content: 'Use your child\'s intense focus on specific interests as an opportunity to extend these interests into social contexts rather than keeping them isolated.',
                  example: 'If your child is fascinated with dinosaurs, take them to children\'s dinosaur clubs or museum activities where they can meet peers who share this interest. Common interests make it easier to connect with others—they might be willing to share dinosaur knowledge or play scenario games with dinosaur toys together.'
                },
                {
                  title: 'Establish Turn-Based Attention Rules',
                  content: 'Teach your child to gradually consider others\' ideas during play activities.',
                  example: 'Implement a "taking turns choosing activities" approach: agree with your child and their playmate that "we\'ll play your favorite game for 20 minutes, then play what your friend wants to do for the next 20 minutes." Initially, your child may be reluctant, but you can enforce this rule with timers and reminders, praising them when they transition to the other child\'s choice.'
                },
                {
                  title: 'Develop Empathy and Awareness of Others',
                  content: 'Address your child\'s tendency to ignore others\' emotions by teaching them to recognize emotional responses.',
                  example: 'Use emotion face cards or picture books to help them identify others\' emotional expressions. For instance, when another child is frowning, show the "sad" emotion card and quietly say, "Look, Jamie is frowning. He might be feeling unhappy. Shall we ask him what\'s wrong?" This reminds your child to notice peers\' expressions and respond accordingly.'
                },
                {
                  title: 'Guide Sharing and Discussion',
                  content: 'When your child is immersed in talking about their favorite topic, gently interrupt to teach question-asking and listening skills.',
                  example: 'If they\'re talking non-stop about dinosaurs, prompt them: "Why don\'t you ask your friend which dinosaur they like best?" If they follow your suggestion and listen quietly to the answer, acknowledge this with an approving nod. This guidance helps them understand that social interaction is two-way, requiring turn-taking in conversation.'
                }
              ]
            }
          ]
        },
        {
          title: 'Communication Issues',
          question: 'When your child wants to express needs or interests, they typically:',
          options: [
            { 
              text: 'Rarely uses language or other clear methods to express needs (poor non-verbal communication)',
              recommendations: [
                {
                  title: 'Introduce Alternative Communication Methods',
                  content: 'If your child struggles with verbal expression, teach them other ways to communicate their needs, such as gestures and picture communication.',
                  example: 'Start by having your child point or take your hand to lead you to desired items. You can also use a Picture Exchange Communication System (PECS): prepare picture cards of common items (cup, food, bathroom), and teach your child to give you the appropriate picture when they need something.'
                },
                {
                  title: 'Provide Choices Instead of Open Questions',
                  content: 'Give your child opportunities to make choices between two options to express preferences.',
                  example: 'Instead of asking "What do you want to drink?" show two options, like milk and juice, and ask "Do you want milk or juice?" This makes communication more concrete and manageable.'
                }
              ]
            },
            { 
              text: 'Has some language ability but mostly repeats limited words or phrases (echolalia)',
              recommendations: [
                {
                  title: 'Recognize and Respond to Signals',
                  content: 'Pay attention to your child\'s non-verbal signals, such as when they take your hand to the kitchen when hungry, or lingering by the door when wanting to go outside.',
                  example: 'When you notice these behaviors, interpret them aloud: "You\'re pulling me to the kitchen. Are you hungry?" Then immediately respond appropriately (offering food while naming it). This "interpret and respond" approach helps children feel understood and learn to use language for needs.'
                },
                {
                  title: 'Build Communication Networks',
                  content: 'Create a supportive environment where all communication attempts are recognized and reinforced.',
                  example: 'Ensure that all caregivers recognize your child\'s unique communication signals and respond consistently, helping to build a reliable communication network across different settings and people.'
                }
              ]
            }
          ]
        },
        {
          title: 'Emotional and Behavioral Management',
          question: 'When facing frustration or disappointment, your child typically:',
          options: [
            { 
              text: 'Extreme emotional outbursts, crying or screaming (poor emotional self-regulation)',
              recommendations: [
                {
                  title: 'Teach Emotion Recognition',
                  content: 'When your child is calm, spend time teaching a few simple emotional vocabulary words with visual cues.',
                  example: 'Teach your child how to identify basic emotional states (e.g., "I\'m happy" or "I\'m sad"). Collect several "feeling faces" cards and practice identifying emotions together. You can also have them point to the face on an emotion chart that best matches how they feel, such as a "smiley face." This helps them recognize and describe their emotions, which is the first step to managing them.'
                },
                {
                  title: 'Set Boundaries While Respecting the Child',
                  content: 'Provide clear emotional boundaries and positive behavior expectations for your child, discussing them after they calm down.',
                  example: 'When your child becomes upset, calmly state: "I understand you\'re frustrated, but hitting is not okay. We can talk about what you need when you\'re calm." This acknowledges their feelings while setting clear limits.'
                }
              ]
            },
            { 
              text: 'Withdraws or avoids situations that cause discomfort (poor emotional coping skills)',
              recommendations: [
                {
                  title: 'Create a "Calm Corner"',
                  content: 'Set up a dedicated space at home where your child can go to find calm when emotions are overwhelming.',
                  example: 'Fill the calm corner with comforting items (pillows, blankets, stuffed animals) and sensory-soothing tools like noise-canceling headphones, stress toys, or bubble tubes. When you notice signs of your child becoming overwhelmed, gently guide them: "I see you\'re feeling uncomfortable, would you like to sit in the quiet area for a bit?" This teaches your child to actively seek emotional balance.'
                },
                {
                  title: 'Develop Emotional Vocabulary',
                  content: 'Help your child identify and name their emotions to better manage them.',
                  example: 'Use emotion cards, books, or apps with pictures showing different feelings. Regularly discuss emotions during calm moments: "The character in this story looks sad. How can you tell? What makes you feel sad sometimes?"'
                }
              ]
            }
          ]
        },
        {
          title: 'Sensory Sensitivity Issues',
          question: 'Which type of sensory input does your child seem most sensitive or strongly reactive to?',
          options: [
            { 
              text: 'Auditory sensitivity (becomes agitated or confused in noisy environments or with harsh sounds)',
              recommendations: [
                {
                  title: 'Provide Auditory Protection',
                  content: 'To reduce noise impact on your child, offer headphones or earmuffs that block or reduce sound in noisy environments.',
                  example: 'Use noise-canceling headphones in crowded, noisy places like shopping malls, or noise-reducing earmuffs at school. These tools help reduce auditory overload by appropriately filtering external sounds, preventing emotional meltdowns due to sensory overstimulation.'
                },
                {
                  title: 'Sound Adaptation Training',
                  content: 'Complete avoidance of noise isn\'t realistic; gradually build your child\'s tolerance to sounds.',
                  example: 'Create a home sound library with recordings of different sounds (vacuum cleaner, blender, crowd noise), starting with very low volume and gradually increasing as your child becomes comfortable. Practice for 5-10 minutes several times weekly.'
                }
              ]
            },
            { 
              text: 'Tactile sensitivity (abnormal reactions to touch, refusing to be touched)',
              recommendations: [
                {
                  title: 'Provide Quiet Retreats',
                  content: 'Ensure your child knows where to find quiet spaces to relieve pressure when overwhelmed.',
                  example: 'Create a small quiet corner at home with their favorite soft toys and draped fabric. When environments become overwhelming, they can seek out this designated quiet space as a safe haven. At school, communicate with teachers about your child\'s need for occasional quiet breaks, helping the teacher adjust their approach.'
                },
                {
                  title: 'Use Deep Pressure Touch',
                  content: 'Many children with tactile sensitivity respond well to firm, predictable touch rather than light touch.',
                  example: 'Offer weighted blankets, compression vests, or tight-fitting undergarments that provide consistent pressure. Practice "sandwich hugs" where you apply even pressure from both sides, which can be more tolerable than light or unexpected touches.'
                }
              ]
            }
          ]
        }
      ]
    };
  },
  methods: {
    showRecommendation() {
      this.currentSelectedOption = this.scenarios[this.currentScenario].options[this.selected];
      // Store the selection with index to ensure correct order
      if (this.userSelections.length <= this.currentScenario) {
        this.userSelections.push({
          scenarioTitle: this.scenarios[this.currentScenario].title,
          scenarioQuestion: this.scenarios[this.currentScenario].question,
          optionText: this.currentSelectedOption.text,
          recommendations: this.currentSelectedOption.recommendations,
          scenarioIndex: this.currentScenario // Add index to track original scenario
        });
      } else {
        this.userSelections[this.currentScenario] = {
          scenarioTitle: this.scenarios[this.currentScenario].title,
          scenarioQuestion: this.scenarios[this.currentScenario].question,
          optionText: this.currentSelectedOption.text,
          recommendations: this.currentSelectedOption.recommendations,
          scenarioIndex: this.currentScenario // Add index to track original scenario
        };
      }
      // Smooth scroll to top
      window.scrollTo({ top: 0, behavior: 'smooth' });
    },
    resetCurrentScenario() {
      this.selected = null;
      this.currentSelectedOption = null;
      // Smooth scroll to top
      window.scrollTo({ top: 0, behavior: 'smooth' });
    },
    navigateToScenario(index) {
      this.currentScenario = index;
      this.selected = null;
      this.currentSelectedOption = null;
      // Smooth scroll to top
      window.scrollTo({ top: 0, behavior: 'smooth' });
    },
    navigateToNextScenario() {
      if (this.currentScenario < this.scenarios.length - 1) {
        this.currentScenario++;
        this.selected = null;
        this.currentSelectedOption = null;
        // Smooth scroll to top
        window.scrollTo({ top: 0, behavior: 'smooth' });
      }
    },
    completeScenarios() {
      // Show results page instead of alert
      this.showResults = true;
    },
    resetAllScenarios() {
      this.currentScenario = 0;
      this.selected = null;
      this.currentSelectedOption = null;
      this.userSelections = [];
      window.scrollTo({ top: 0, behavior: 'smooth' });
    },
    backToScenarios() {
      this.showResults = false;
      this.resetAllScenarios();
    },
    downloadPDF() {
      // Create a new jsPDF instance
      import('jspdf').then(({ default: jsPDF }) => {
        import('html2canvas').then(({ default: html2canvas }) => {
          const doc = new jsPDF('p', 'pt', 'a4');
          const content = document.getElementById('results-content');
          const pageWidth = doc.internal.pageSize.getWidth();
          
          // Add title
          doc.setFont('helvetica', 'bold');
          doc.setFontSize(18);
          doc.setTextColor(81, 122, 57);
          doc.text('Personalized Autism Support Recommendations', 40, 40);
          
          // Add intro text
          doc.setFontSize(12);
          doc.setFont('helvetica', 'normal');
          doc.setTextColor(0);
          doc.text('These recommendations are tailored to your child\'s specific needs. Consult with professionals for personalized guidance.', 40, 70, { maxWidth: pageWidth - 80 });
          
          // Better approach for capturing content
          const options = {
            scale: 1.5,
            useCORS: true,
            allowTaint: true,
            logging: false,
            letterRendering: true,
            width: content.offsetWidth,
            height: content.offsetHeight
          };
          
          html2canvas(content, options).then(canvas => {
            // Adjust PDF to properly fit content
            const imgData = canvas.toDataURL('image/png');
            const imgWidth = doc.internal.pageSize.getWidth() - 80; // Add margins
            const pageHeight = doc.internal.pageSize.getHeight();
            const imgHeight = (canvas.height * imgWidth) / canvas.width;
            let heightLeft = imgHeight;
            let position = 100; // Start position after title
            
            // Add first page
            doc.addImage(imgData, 'PNG', 40, position, imgWidth, imgHeight);
            heightLeft -= pageHeight - position;
            
            // Add new pages if content overflows
            while (heightLeft > 0) {
              position = heightLeft - imgHeight;
              doc.addPage();
              doc.addImage(imgData, 'PNG', 40, position, imgWidth, imgHeight);
              heightLeft -= pageHeight;
            }
            
            // Add footer on last page
            doc.setFontSize(10);
            doc.setTextColor(100);
            const lastPage = doc.internal.getNumberOfPages();
            doc.setPage(lastPage);
            doc.text('Generated by Supporting Autism Families - ' + new Date().toLocaleDateString(), 40, doc.internal.pageSize.getHeight() - 20);
            
            // Save the PDF
            doc.save('personalized-autism-support-recommendations.pdf');
          });
        });
      }).catch(error => {
        console.error('PDF generation error:', error);
        alert('Unable to generate PDF. Using browser print instead.');
        window.print();
      });
    },
    formatRecommendations(recommendations) {
      // If already in object array format, return directly
      if (recommendations.length > 0 && typeof recommendations[0] === 'object') {
        return recommendations;
      }
      
      // If string array, convert to object format
      return recommendations.map(rec => {
        // Try to split string to get title and content
        const parts = rec.split(':');
        if (parts.length > 1) {
          return {
            title: parts[0],
            content: parts[1],
            example: parts.length > 2 ? parts[2] : 'Try following the recommendations and observe your child\'s response, adjusting as needed for the best results.'
          };
        }
        
        // If can't split, return entire string as content
        return {
          title: this.generateSimpleTitle(rec),
          content: rec,
          example: 'Try following the recommendations and observe your child\'s response, adjusting as needed for the best results.'
        };
      });
    },
    generateSimpleTitle(recommendation) {
      // Generate a simple title from recommendation content
      const firstSentence = recommendation.split('.')[0];
      if (firstSentence.length > 15) {
        return firstSentence.substring(0, 15) + '...';
      }
      return firstSentence;
    }
  }
};
</script>

<style scoped>

.fullscreen-container {
  background-color: #F8EFED;
  min-height: 100vh;
  width: 100vw;
  overflow-x: hidden;
  display: flex;
  flex-direction: column;
}

.sidebar-container {
  position: relative;
}

.sidebar-content {
  top: 0;
  padding: 2rem;
}

.page-title {
  color: #4d2f20;
  font-size: 2.5rem;
  font-weight: bold;
}

.page-subtitle {
  color: #4d2f20;
  font-size: 1.2rem;
}

.main-image-container {
  text-align: center;
  padding: 0 1rem;
  transition: all 0.5s ease;
}

.main-image {
  border-radius: 15px;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
  max-height: 60vh;
  object-fit: cover;
  transition: transform 0.5s ease;
}

.progress-container {
  margin-top: 1rem;
}

.progress-text, .progress-percentage {
  color: #4d2f20;
  font-weight: 600;
}

.progress {
  height: 8px;
  background-color: #E9E9E9;
}

.progress-bar {
  background-color: #3E5C2B;
  transition: width 0.5s ease;
}

.content-card {
  background-color: #F8EFED;
  border-radius: 15px;
  margin: 2rem;
}

.scenario-title {
  color: #4d2f20;
  font-size: 1.2rem;
  font-weight: 500;
}

.scenario-question {
  color: #4d2f20;
  font-size: 1.8rem;
  font-weight: 600;
}

.option-card {
  border-left: 4px solid #3E5C2B;
  background-color: #FFFFFF;
  border-radius: 8px;
  transition: all 0.3s ease;
  font-size: 1.2rem;
  color: #333333;
  position: relative;
  overflow: hidden;
  cursor: pointer;
}

.option-card:hover {
  transform: translateX(5px);
  background-color: #F7F0ED;
  box-shadow: 0 4px 8px rgba(77, 47, 32, 0.1);
}

.selected-option {
  background-color: #F7F0ED;
  transform: translateX(5px);
  box-shadow: 0 4px 8px rgba(77, 47, 32, 0.1);
}

.option-indicator {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  border: 2px solid #4d2f20;
  flex-shrink: 0;
  position: relative;
}

.selected-indicator::after {
  content: '';
  position: absolute;
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background-color: #3E5C2B;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.option-text {
  color: #4d2f20;
}

.action-button {
  background-color: #3E5C2B;
  color: white;
  font-size: 1.2rem;
  border-radius: 8px;
  transition: all 0.3s ease;
  border: none;
}

.action-button:hover:not(:disabled) {
  background-color: #3E5C2B;
  box-shadow: 0 4px 10px rgba(77, 47, 32, 0.3);
  transform: translateY(-2px);
}

.action-button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.back-button {
  background-color: transparent;
  color: #3E5C2B;
  border: 2px solid #3E5C2B;
  font-size: 1.1rem;
  border-radius: 8px;
  transition: all 0.3s ease;
  padding: 10px 20px;
}

.back-button:hover {
  background-color: rgba(77, 47, 32, 0.1);
  transform: translateY(-2px);
}

.next-button, .finish-button {
  background-color: #3E5C2B;
  color: white;
  font-size: 1.1rem;
  border-radius: 8px;
  transition: all 0.3s ease;
  padding: 10px 20px;
  border: none;
}

.next-button:hover, .finish-button:hover {
  background-color: #3E5C2B;
  box-shadow: 0 4px 10px rgba(77, 47, 32, 0.3);
  transform: translateY(-2px);
}

.scenario-dot {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background-color: #D8D8D8;
  cursor: pointer;
  transition: all 0.3s ease;
}

.scenario-dot.active {
  background-color: #3E5C2B;
}

.scenario-dot.current {
  transform: scale(1.3);
  box-shadow: 0 0 0 3px rgba(77, 47, 32, 0.2);
}

.recommendations-heading {
  color: #4d2f20;
  font-weight: 600;
}

.recommendation-content {
  background-color: #FFFFFF;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
}

.recommendation-subtitle {
  color: #3E5C2B;
  font-weight: 600;
}

.recommendation-cards {
  max-height: 70vh;
  overflow-y: auto;
  padding-right: 5px;
}

.recommendation-card {
  transition: all 0.3s ease;
  border-radius: 8px;
  overflow: hidden;
}

.recommendation-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.08);
}

.card-header {
  background-color: #F7F0ED;
  border-radius: 8px 8px 0 0;
  cursor: pointer;
}

.rec-icon {
  color: #3E5C2B;
  font-size: 1.2rem;
  flex-shrink: 0;
}

.recommendation-title {
  color: #3E5C2B;
  font-weight: 600;
  font-size: 1.1rem;
}

.card-body .content {
  font-size: 1rem;
  line-height: 1.5;
  color: #3E5C2B;
}

.example {
  background-color: #F8F8F8;
  border-left: 3px solid #3E5C2B;
  border-radius: 4px;
  font-style: italic;
  color: #4d2f20;
}

.example-title {
  font-size: 0.9rem;
  font-weight: 500;
  color: #4d2f20;
}

.example-content {
  font-size: 0.9rem;
  color: #4d2f20;
}

.results-container {
  max-width: 1000px;
  margin: 0 auto;
}

.results-card {
  background-color: #FFFFFF;
  border-radius: 15px;
}

.results-title {
  color: #4d2f20;
  font-size: 2.2rem;
  font-weight: bold;
}

.results-intro {
  background-color: #F7F0ED;
  border-radius: 10px;
}

.results-intro-title {
  color: #4d2f20;
}

.scenario-number {
  background-color: #3E5C2B;
  color: white;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  margin-right: 15px;
}

.scenario-result-title {
  color: #3E5C2B;
  font-weight: 600;
  margin-bottom: 0;
}

.scenario-result-question {
  color: #3E5C2B;
}

.selected-option-box {
  background-color: #F8F8F8;
  border-left: 4px solid #3E5C2B;
  border-radius: 8px;
}

.selected-option-title {
  color: #3E5C2B;
  font-weight: 600;
}

.recommendations-summary-title {
  color: #3E5C2B;
  font-weight: 600;
  margin-bottom: 15px;
}

.recommendation-item {
  background-color: #FFFFFF;
  border: 1px solid #E0E0E0;
  border-radius: 8px;
  transition: all 0.3s ease;
}

.recommendation-item:hover {
  box-shadow: 0 4px 12px rgba(77, 47, 32, 0.1);
}

.recommendation-item-title {
  color: #3E5C2B;
  font-weight: 600;
}

.download-button {
  background-color: #3E5C2B;
  color: white;
  border-radius: 8px;
  transition: all 0.3s ease;
  padding: 10px 20px;
  border: none;
}

.download-button:hover {
  background-color: #3E5C2B;
  box-shadow: 0 4px 10px rgba(77, 47, 32, 0.3);
  transform: translateY(-2px);
}

/* 滚动条样式 */
.recommendation-cards::-webkit-scrollbar {
  width: 8px;
}

.recommendation-cards::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 10px;
}

.recommendation-cards::-webkit-scrollbar-thumb {
  background: #3E5C2B;
  border-radius: 10px;
}

.recommendation-cards::-webkit-scrollbar-thumb:hover {
  background: #5a873f;
}

/* 动画 */
.recommendation-container {
  animation: fadeIn 0.5s;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

/* 响应式调整 */
@media (max-width: 768px) {
  .row {
    flex-direction: column;
  }
  
  .main-image {
    max-height: 40vh;
    margin-bottom: 2rem;
  }
  
  .recommendation-cards {
    max-height: 50vh;
  }
}

/* 打印样式 */
@media print {
  .download-button, .back-button {
    display: none;
  }
  
  .results-container {
    padding: 0;
    max-width: 100%;
  }
  
  .results-card {
    box-shadow: none !important;
    border: none !important;
  }
  
  body {
    font-size: 12pt;
  }
  
  h1 {
    font-size: 18pt !important;
  }
  
  h2 {
    font-size: 16pt !important;
  }
  
  h3 {
    font-size: 14pt !important;
  }
}
</style>
