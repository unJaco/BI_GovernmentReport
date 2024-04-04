<script>
    import Button from "./Button.svelte"

    export let metric
    export let value

    let apiKey = ''

    let variable = 'Want an explanation? Press the explain button...'
    
    let request = ''
    function makeGPTCall() {
        console.log()
        request = 'The correlation between "Trust in National Government" and the ' + metric + ' is ' + value
        console.log(request)
        fetch('https://api.openai.com/v1/chat/completions', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ` + apiKey
            },
            body: JSON.stringify({
                model: 'gpt-3.5-turbo-0125', // Specify the model you want to use
                messages: [
                    {
                        "role" : "system",
                        "content" : "You are a data analyst in the field of politics. You will be asked about the correlation between certain categories. Write one sentence what this correltation means. And one sentence why you think this correlation is the way it is!"
                    },
                    {
                        "role" : "user",
                        "content" : request
                    },
                    
                 ],
                temperature: 0.7,
                max_tokens: 200,
            })
            })
            .then(response => response.json())
            .then(data => {
                variable = data.choices[0].message.content
                console.log("SUCCESS")  
            })
            .catch(error => variable = 'Please insert a valid API Key!')
    }
</script>




<style>
    .container {
        display: flex;
        justify-content: flex-end; /* Aligns the child elements to the right */
        align-items: center; /* Aligns the child elements vertically */
    }
  
    .paragraph-border {
        border: 2px solid #ddd; /* Example border style */
        padding: 10px;
        margin-top: 10px; /* Adjust based on your design */
        border-radius: 10px; /* Adds rounded corners */
    }

  </style>
  
  <div class="container">
    <Button
      class="font-ui font-normal text-base no-underline border mt-5 rounded-lg py-2 px-3 inline-block transition hover:ease-in hover:border-blue-200 hover:bg-blue-100 hover:no-underline hover:text-gray-800 focus:text-gray-900 focus:border-blue-200"
      on:click={makeGPTCall}
    >
      Explain
    </Button>
  </div>
  <div class="paragraph-border">
    <p>{variable}</p>
  </div>
  
