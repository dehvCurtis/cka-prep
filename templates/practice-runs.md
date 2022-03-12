# Practice Runs

## Description

This page is used to perform repetative practice for everything that you've learned. 

This marathon is written from an imperative perspective, since the CKA is a timed test.

## Marathon

### Workloads and Scheduling

1) Create deployment

   1) <details>
        <summary>solution</summary>
        kubectl create deployment {deployment-name} --image={image-name}
      </details>

2) Scale up deployment

   1) <details>
        <summary>solution</summary>
        kubectl scale deployment {deployment-name} --replicas={scale-count}
      </details>

3) Scale down deployment

   1) <details>
        <summary>solution</summary>
        kubectl scale deployment {deployment-name} --replicas={scale-count}
      </details>

4) Rolling Update

   1) <details>
        <summary>solution</summary>
        kubectl set image deployment test-deployment {name-of-container}={new-image-name}
      </details>

5) Rolling Back Deployment

   1) <details>
        <summary>solution</summary>
        conten
      </details>