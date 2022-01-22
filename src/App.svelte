<script lang="ts">
  import { onMount } from "svelte";
  import { Connection } from "@solana/web3.js";
  import { Provider, web3 } from "@project-serum/anchor";
  import { fade } from "svelte/transition";
  import Button from "./components/Button.svelte";
  import Header from "./components/CardHeader.svelte";

  import { candyMachineState, userState } from "./lib/store";
  import {
    getCandyMachineState,
    checkWalletConnected,
    getUserBalance,
    existsOwnerSPLToken,
  } from "./lib/state-helpers";

  /***********************************/
  // Customise the app by changing the following variables.
  const TITLE = "Solana HODL Whales";
  const DESCRTIPTION = "BREED | HODL | EARN";
  const HEADER_TITLE = "";
  const HEADER_LINK = "";
  // Your image or GIF needs to be in the /public folder for this to work
  const IMAGE_LINK = "https://cdn.discordapp.com/attachments/930785564377088000/934504136135823440/slowgif.gif" ;
  /***********************************/

  const { solana } = window as any;
  const rpcUrl = import.meta.env.VITE_APP_SOLANA_RPC_HOST?.toString();
  const cluster = import.meta.env.VITE_APP_SOLANA_NETWORK?.toString();
  const candyMachineId = import.meta.env.VITE_APP_CANDY_MACHINE_ID?.toString();
  const opts = { preflightCommitment: "processed" };

  let siteLoading = true;
  let errorOcurred = false;
  let connection: Connection;
  let provider: Provider;
  let candyMachinePublicKey: web3.PublicKey;

  $: itemsRedeemed = $candyMachineState?.state.itemsRedeemed;
  $: itemsAvailable = $candyMachineState?.state.itemsAvailable;

  function checkEnvironmentVariables() {
    // Check if populated
    if (!rpcUrl || !candyMachineId || !cluster) {
      if (!rpcUrl) {
        console.error("RPC URL not populated");
      }
      if (!candyMachineId) {
        console.error("Candy Machine ID not populated");
      }
      if (!cluster) {
        console.error("Environment not populated");
      }
      return true;
    }
    if (candyMachineId.length < 32 || candyMachineId.length > 44) {
      console.error(
        "Candy Machine Public Key is invalid. Enter a length in-between 32 and 44 characters"
      );
      return true;
    }
    return false;
  }

  onMount(async () => {
    // Check if environement variables are populated
    errorOcurred = checkEnvironmentVariables();
    if (errorOcurred) {
      return;
    }

    // If env variables populated, create provider, PK and connection
    connection = new Connection(rpcUrl);
    provider = new Provider(
      connection,
      solana,
      opts.preflightCommitment as web3.ConfirmOptions
    );
    candyMachinePublicKey = new web3.PublicKey(candyMachineId);

    // Get candy machine state
    $candyMachineState = await getCandyMachineState(
      candyMachinePublicKey,
      provider
    );

    // Establish connection to wallet
    if (solana?.isPhantom) {
      $userState.walletPublicKey = await checkWalletConnected(solana);
      if ($userState.walletPublicKey) {
        // Get User Balance
        $userState.userBalance = await getUserBalance(
          $userState.walletPublicKey,
          connection
        );
        // Check if user is whitelisted (ie. check if they have token)
        $userState.isWhiteListed = await existsOwnerSPLToken(
          $userState.walletPublicKey,
          connection,
          $candyMachineState.state.whitelistMintSettings?.mint
        );
      }
    }

    // Stop loading
    siteLoading = false;
  });
</script>

<main class="h-screen">
  <!-- Error section -->
  {#if errorOcurred}
    <div class=" h-full flex">
      <div class="m-auto">
        An error occurred. Please check if your environment variables have been
        populated correctly and redeploy the applcation.
      </div>
    </div>
    <!-- Loading Section -->
  {:else if siteLoading && !errorOcurred}
    <div class=" h-full flex">
      <div class="lds-hourglass m-auto" />
    </div>
  {:else}
    <!-- Menu Bar -->   
       <input type="checkbox" id="active">
       <label for="active" class="menu-btn"><i class="fas fa-bars"></i></label>
       <div class="wrapper">
         <ul>
   <li><a href="#">Home</a></li>
   <li><a href="#">About</a></li>
   <li><a href="#">Services</a></li>
   <li><a href="#">Gallery</a></li>
   <li><a href="#">Feedback</a></li>
   </ul>
   </div>
   <div class="content">
         <div class="title">
   </div>
   <p>
   </p>
       </div>
     <link href="https://fonts.googleapis.com/css2?family=Oswald&display=swap" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Lato&display=swap" rel="stylesheet">
       <script src="https://kit.fontawesome.com/a076d05399.js"></script>

   
       <input type="checkbox" id="active">
       <label for="active" class="menu-btn"><i class="fas fa-bars"></i></label>
       <div class="wrapper">
         <ul>
   <li><a href="#">Home</a></li>
   <li><a href="#">About</a></li>
   <li><a href="#">Services</a></li>
   <li><a href="#">Gallery</a></li>
   <li><a href="#">Feedback</a></li>
   </ul>
   </div>
    {#if HEADER_TITLE}
      <a
        href={HEADER_LINK}
        class="text-blue tracking-widest underline underline-offset-4 decoration-2 font-mono"
        >{HEADER_TITLE}</a
      >
    {/if}
    <!-- Card -->
    <div
      class=" max-w-lg mx-auto bg-white rounded-lg my-12  border-2"
      transition:fade
    >
      <!-- Top Bar -->
      <Header />
      <hr />
      <br />
      <!-- Main Body -->
      <div class="p-6">
        <img src={IMAGE_LINK} alt="" class=" w-1/2 mx-auto m-5" style="border-radius: 25%;"/>
        <div
          class=" text-lg sm:text-2xl font-mono font-bold py-5 tracking-wider"
        >
          {TITLE}
        </div>
        <div class="text-sm sm:text-md font-semibold pb-6 text-gray-600 ">
          {DESCRTIPTION}
        </div>
        <Button {solana} {connection} />

        <div class=" tracking-widest font-bold text-sm pt-3 text-gray-400">
          {itemsRedeemed}/{itemsAvailable} claimed
        </div>
        <div class="flex flex-col pt-3">
          {#if $userState.solanaExplorerLink}
            <a
              href={$userState.solanaExplorerLink}
              target="_blank"
              class="text-purple-700 font-semibold  p-1"
              >View on Solana Explorer</a
            >
          {/if}
        </div>
      </div>
    </div>
  {/if}
  
<!-- Site footer -->
<footer class="site-footer">
  <div class="container">
      <div class="row">
          <div class="col-sm-12 col-md-6">
              <h6>About</h6>
              <p class="text-justify">Scanfcode.com <i>CODE WANTS TO BE SIMPLE </i> is an initiative to help the upcoming programmers with the code. Scanfcode focuses on providing the most efficient code or snippets as the code wants to be simple. We will help programmers
                  build up concepts in different programming languages that include C, C++, Java, HTML, CSS, Bootstrap, JavaScript, PHP, Android, SQL and Algorithm.</p>
          </div>

          <div class="col-xs-6 col-md-3">
              <h6>Categories</h6>
              <ul class="footer-links">
                  <li><a href="http://scanfcode.com/category/c-language/">C</a></li>
                  <li><a href="http://scanfcode.com/category/front-end-development/">UI Design</a></li>
                  <li><a href="http://scanfcode.com/category/back-end-development/">PHP</a></li>
                  <li><a href="http://scanfcode.com/category/java-programming-language/">Java</a></li>
                  <li><a href="http://scanfcode.com/category/android/">Android</a></li>
                  <li><a href="http://scanfcode.com/category/templates/">Templates</a></li>
              </ul>
          </div>

          <div class="col-xs-6 col-md-3">
              <h6>Quick Links</h6>
              <ul class="footer-links">
                  <li><a href="http://scanfcode.com/about/">About Us</a></li>
                  <li><a href="http://scanfcode.com/contact/">Contact Us</a></li>
                  <li><a href="http://scanfcode.com/contribute-at-scanfcode/">Contribute</a></li>
                  <li><a href="http://scanfcode.com/privacy-policy/">Privacy Policy</a></li>
                  <li><a href="http://scanfcode.com/sitemap/">Sitemap</a></li>
              </ul>
          </div>
      </div>
      <hr>
  </div>
  <div class="container">
      <div class="row">
          <div class="col-md-8 col-sm-6 col-xs-12">
              <p class="copyright-text">Copyright &copy; 2017 All Rights Reserved by
                  <a href="#">Scanfcode</a>.
              </p>
          </div>

          <div class="col-md-4 col-sm-6 col-xs-12">
              <ul class="social-icons">
                  <li><a class="facebook" href="#"><i class="fa fa-facebook"></i></a></li>
                  <li><a class="twitter" href="#"><i class="fa fa-twitter"></i></a></li>
                  <li><a class="dribbble" href="#"><i class="fa fa-dribbble"></i></a></li>
                  <li><a class="linkedin" href="#"><i class="fa fa-linkedin"></i></a></li>
              </ul>
          </div>
      </div>
  </div>
</footer>

</main>
