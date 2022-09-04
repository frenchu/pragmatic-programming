+++
date = 2022-04-02T17:34:31+02:00
title = "Affordable cloud solutions for programmers"
description = "VPS comparison"
tags = ["vps", "digitalocean", "linode", "vultr", "upcloud"]
categories = ["devops", "cloud"]
+++

## Introduction

## Results

### CPU performance

{{< chart 150 400 >}}
{
  type: 'bar',
  data: {
    labels: [ 'Compressing', 'Decompressing', 'Overall' ],
    datasets: [
      {
        label: 'DigitalOcean',
        data: [ 11529, 11258, 11393 ],
        borderColor: 'rgba(54, 162, 235, 1)',
        backgroundColor: 'rgba(54, 162, 235, 0.6)'
      },
      {
        label: 'Linode',
        data: [ 9738, 11948, 10843],
        borderColor: 'rgba(255, 99, 132, 1)',
        backgroundColor: 'rgba(255, 99, 132, 0.6)'
      },
      {
        label: 'Vultr',
        data: [ 16615, 14181, 15398 ],
        borderColor: 'rgba(255, 206, 86, 1)',
        backgroundColor: 'rgba(255, 206, 86, 0.6)'
      },
      {
        label: 'UpCloud',
        data: [ 16503, 14521, 15512 ],
        borderWidth: 2,
        borderColor: 'rgba(75, 192, 192, 1)',
        backgroundColor: 'rgba(75, 192, 192, 0.6)'
      }
    ]
  },
  options: {
    responsive: true,
    maintainAspectRatio: false,
    datasets: {
      bar: {
        borderWidth: 2
      }
    },
    scales: {
      y: {
        title: {
          display: true,
          text: 'MIPS'
        }
      }
    },
    plugins: {
      legend: {
        position: 'right',
      },
      title: {
        display: true,
        text: 'CPU - 7z Test'
      }
    }
  }
}
{{< /chart >}}