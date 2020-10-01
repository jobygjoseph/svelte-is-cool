<script>
  import Grid from "svelte-grid-responsive";

  import AIMS_FLAVORS from "./aimsFlavors";

  export let flavor = [];
  const [flKey, flVal] = flavor;
  export let data = {};

  const isCloudinaryImage = (url) => {
    if (typeof url !== "string") {
      return false;
    }

    return url.includes("media-cldnry") || url.includes("cloudinary.com");
  };

  const getCloudinaryImageSrc = (originalSource, flavor) =>
    originalSource.replace(
      "/image/upload/",
      `/image/upload/t_${flavor},f_auto,q_auto/`
    );

  // Checks that the URL contains formats that need to be converted
  const IS_IMAGE_REGEX = /(.*?)\/(i|j)\/(.*)\/(.*)(jpg|jpeg|gif|png|bmp)$/i;

  const isImage = (url) => !!url.match(IS_IMAGE_REGEX);

  const getAimsSignatureForFlavor = (flavor, { width, height }) => {
    switch (flavor) {
      case AIMS_FLAVORS.FIT:
        return `${flavor}-${width}w`;
      case AIMS_FLAVORS.FOCAL:
        return `${flavor}-${width}x${height}`;
      default:
        return flavor;
    }
  };

  function getAimsUrlFlavor(
    url,
    { width = 800, height = 320, flavor = AIMS_FLAVORS.FOCAL }
  ) {
    if (!url || !isImage(url)) {
      return null;
    }

    const signature = getAimsSignatureForFlavor(flavor, { width, height });

    return url
      .replace(/^http:/, "https:")
      .replace(/\/i\//, "/j/")
      .replace(/\.(jpg|jpeg|png|gif|bmp)/i, `.${signature}.$1`);
  }

  // https://nbcnewsdigital.atlassian.net/wiki/spaces/NEWSDEVICE/pages/38600882/AIMS+Developer+Guide
  function getAimsUrlSuffix(
    url,
    { width = 640, height = 480, resample = 7, compression = 70, mode = 3 }
  ) {
    if (!url || !isImage(url)) {
      return null;
    }

    // Resampling algorithm is a value from 0 to 7. Use 7 for best results.
    const r = [1, 2, 3, 4, 5, 6, 7].indexOf(resample) >= 0 ? resample : 7;

    // Compression ratio is a value from 0 to 100.
    const c =
      !Number.isNaN(compression) && compression >= 0 && compression <= 100
        ? compression
        : 70;

    // 1 - Box (scales to fit, then centers and adds black bars to fill the rest of the space)
    // 2 - Center crop
    // 3 - Scale to fit
    // 4 - T-crop (scales to fill and centers on the upper portion of the image)
    // 5 - Smart T-crop (crops to the subject of the image, such as a person's face)
    const m = [1, 2, 3, 4, 5].indexOf(mode) >= 0 ? mode : 3;

    const signature = `${width};${height};${r};${c};${m}`;

    return url
      .replace(/^http:/, "https:")
      .replace(/\/i\//, "/j/")
      .replace(/\.(jpg|jpeg|png|gif|bmp)/i, `.${signature}.$1`);
  }

  const clImg = data?.CloudinaryImage?.url?.primary;
  const aimImg = data?.AimsImage?.url?.primary;

  const clSrc = getCloudinaryImageSrc(clImg, flKey);
  const aimSrc = getAimsUrlFlavor(aimImg, { width: 500, flavor: flVal });
</script>

<style>
  img {
    max-width: 100%;
  }
</style>

<Grid xs={2} md={1} lg={1}><strong>{flKey}</strong></Grid>

<Grid xs={2} md={1} lg={1}><img src={clSrc} alt={flKey} /></Grid>

<Grid xs={2} md={1} lg={1}><img src={aimSrc} alt={flKey} /></Grid>
