# Lab 09: Knowledge check

**Answer each question below it, in your own words, in one to three sentences.** Replace the `...` below **Your answer:** with your text.

## 1. Credentials in the image?

Assume you want to share your image with others and make it publicly available on `ghcr.io`, so that anyone can pull it and run the analysis. A teammate suggests adding `COPY .aws /home/app/.aws` to the `Dockerfile`, so that nobody has to type `-v ~/.aws:/home/app/.aws:ro` any more. (This is a thought experiment: keep your own package private.)

**Your answer:** Give two reasons why this is a bad idea. Think of who could then read your credentials, and of what happens when your Learner Lab session ends.

Anyone who pulls the public image could extract the AWS credentials from its layers, even if a later layer deleted the copied files. Learner Lab credentials also expire when the session ends, so embedding them would make the image both unsafe and quickly unusable; mounting current credentials at run time avoids both problems.

## 2. Where did the Atlas password come from?

In Step 4, you ran the analysis once with `-v ~/.aws:/home/app/.aws:ro` only, without the three `-e` options. The container still wrote to your Atlas cluster.

*Hint:* Read `get_atlas_settings()` in `src/book_analysis/analyze.py`.

**Your answer:** Explain, step by step, where the container found the Atlas settings in that run.

Because `MONGODB_ATLAS_URL` was absent, `get_atlas_settings()` used boto3 to request the `mongodb_credentials` parameter from AWS Systems Manager Parameter Store in `us-east-1`, with decryption enabled. Boto3 read the AWS credentials from the read-only mounted `.aws` directory, then the program parsed the parameter's JSON value to obtain the Atlas URL, user, and password.

## 3. Same tag, different image

Run `docker images book-analysis`, and note the image ID of `book-analysis:0.1.0`. Then change the `description` in the `LABEL` of your `Dockerfile`, build again with the same command, `docker build -t book-analysis:0.1.0 .`, and run `docker images book-analysis` again.

**Your answer:** Paste the image ID from before and after the rebuild. Why is it a problem for someone who uses your image that `0.1.0` now names a different image, and what should you have done instead?

Before rebuild: `3cdf028a41aa`. After rebuild: `eb55896bb9ac`. I changed the description and rebuilt, but the tag was still `0.1.0`. That could confuse people because the same tag now points to a different image. I should use a new tag, such as `0.1.1`, for the changed image.
